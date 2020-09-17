---
title: 潘家园综合业务管理系统2
date: 2020-07-24 11:00:00
keys: #密码
- '20729edcf2c71f4714309b8b64a3980c'
publish: true #是否公布
---

## 商户规则
### 在经营商户
### 新入市商户
### 退市商户
### 续签商户



## 合同规则

### 录入合同

### 续签合同

### 终止合同


## 报表规则

### 租赁合同签约情况分析

#### 合同签约情况占比分析

**续签**
- 查询时间范围内，非续签且不是作废的合同

**新入市**
- 查询时间范围内，未终止的，并且原合同id为空的合同（当前逻辑-错误）
- 查询时间范围内，所有原合同id为空并且终止状态为非作废的合同（正确逻辑）

**退市**
- 查询时间范围内，终止的，没有续签的合同


### 场地统计报表规则
#### 场地统计
1. **目的**
- 统计各区及其各区下排的摊位面积及其摊位数量
2. **涉及表**
- 摊位表（biz_store_stall）
3. **显示值**
- 摊位名称(text)
- 总面积(mj)
- 总摊位数(total)
4. **筛选条件**
- 统计各区及其各区下的摊位面积及其摊位数量
5. **Sql脚本**
```sql
SELECT
	* 
FROM
	(
	SELECT
		SUBSTR( id_, 1, 5 ) AS stall_id,
		parent_id,
		text,
		text AS texts,
		sum( IFNULL( jymj, 0 ) ) AS mj,
		count( * ) AS stall_count,
		SUBSTR( id_, 1, 12 ) AS id 
	FROM
		biz_store_stall 
	WHERE
		LENGTH( id_ ) != 2 
	GROUP BY
		SUBSTR( id_, 1, 5 ) UNION ALL
	SELECT
		qu.id_ AS stall_id,
		qu.parent_id,
		qu.text AS texts,
		qu.text AS text,
		stall.mj AS mj,
		stall.stall_count AS stall_count,
		SUBSTR( id_, 1, 12 ) AS id 
	FROM
		(
			( SELECT * FROM biz_store_stall WHERE LENGTH( id_ ) = 2 ) AS qu
			LEFT JOIN (
			SELECT
				SUBSTR( id_, 1, 2 ) AS stall_id,
				sum( IFNULL( jymj, 0 ) ) AS mj,
				count( * ) AS stall_count,
				SUBSTR( id_, 1, 12 ) AS id 
			FROM
				biz_store_stall 
			WHERE
				store_type = 'hao' 
			GROUP BY
				SUBSTR( id_, 1, 2 ) 
			) AS stall ON qu.id_ = stall.stall_id 
		) 
	) AS stall 
ORDER BY
	stall_id;
```
#### 摊位面积占比分析
1. **需求目的**
- 统计各区下的摊位类型为号的摊位面积和的占比
2. **涉及表**
- 摊位表（biz_store_stall）
3. **显示值**
- 摊位名称(adress)
- 摊位面积(Total)
4. **筛选条件**
- 统计各区下的摊位类型为号的摊位面积和的占比
5.  **Sql脚本**
```sql
SELECT
	SUM( jymj ) AS Total,
	'西区' AS adress 
FROM
	biz_store_stall 
WHERE
	SUBSTR( id_, 1, 2 ) = '02' 
	AND store_type = 'hao' UNION
SELECT
	SUM( jymj ) AS Total,
	'东区' AS adress 
FROM
	biz_store_stall 
WHERE
	SUBSTR( id_, 1, 2 ) = '01' 
	AND store_type = 'hao' UNION
SELECT
	SUM( jymj ) AS Total,
	'坐店' AS adress 
FROM
	biz_store_stall 
WHERE
	SUBSTR( id_, 1, 2 ) = '03' 
	AND store_type = 'hao' UNION
SELECT
	SUM( jymj ) AS Total,
	'其它' AS adress 
FROM
	biz_store_stall 
WHERE
	SUBSTR( id_, 1, 2 ) = '10' 
	AND store_type = 'hao';
```

### 业态经营面积占比分析报表规则
#### 当前时间的年月日+业态经营面积占比分析表格
1. **目的**
- 统计查询时间范围内的各业态品类的大类经营面积（jymj）的和、占总面积比
2. **涉及表**
- 摊位表（biz_store_stall）
- 合同表（biz_contract_stall）
- 品类表（biz_cbd_business_projects）    
3. **显示值**
- 摊位表的当前大类经营面积（jymj）的和
- 合同的经营项目大类ID（zyxm）
- 品类表的大类名称（text）
- 占总面积比=当前大类经营面积/所有大类经营面积和
4. **筛选条件**
- 合同摊位id等于摊位id 
- 没有终止的合同
- 合同截止时间大于等于当前时间
- 合同起始时间的年等于当前时间的年
- 根据经营项目分组
5. **Sql脚本**
```sql
SELECT
	* 
FROM
	biz_cbd_business_projects 
WHERE
	layer = 1 
	AND subject_type = '大类';
SELECT
	sum( s.jymj ) 经营面积,
	substring( zyxm, 1, 2 ) 大类 ID,
	( SELECT text FROM biz_cbd_business_projects WHERE id_ = substring( c.zyxm, 1, 2 ) ) 大类名称 
FROM
	biz_contract_stall c,
	biz_store_stall s 
WHERE
	c.stall_id = s.id_ 
	AND sfzz = 'N' 
	AND end_time >= DATE( CURDATE( ) ) 
	AND YEAR ( start_time ) = YEAR ( now( ) ) 
GROUP BY
	substring( zyxm, 1, 2 ) SELECT
	sum( s.jymj ) 经营面积,
	zyxm 小类 ID,
	jyxm 小类名称 
FROM
	biz_contract_stall c,
	biz_store_stall s 
WHERE
	c.stall_id = s.id_ 
	AND sfzz = 'N' 
	AND end_time >= DATE( CURDATE( ) ) 
	AND YEAR ( start_time ) = YEAR ( now( ) ) 
GROUP BY
	zyxm
```

#### 当前时间的年月日+业态经营面积占比分析图
- 取表格中，摊位表的经营面积的和、合同的经营项目的大类ID



### 业态摊位占比分析
#### 当前年月日业态摊位占比分析
1. **目的**
- 统计各经营大类的租户个数和占总数比例
2. **筛选表**
- biz_cbd_business_projects(经营品类表)
- biz_contract_stall(租赁合同表)
3. **显示值**
- 品类编号(id_)
- 品类名称(经营品类名称)
- 租户个数(count(*))
- 占总数比例 =各经营品类租户个数/所以经营品类租户个数
4. **筛选条件**
- sql1：如果有筛选值，那么查询经营品类等于筛选值的数据，否则默认筛选全部
- sql2：筛选没有终止的合同，合同截止时间大于等于当前时间，并且合同起始时间的年份等于当前年份的合同，再根据合同的经营项目大类分组筛选
5.  **sql脚本**
- sql1
```sql
SELECT
	id_,
	text 
FROM
	biz_cbd_business_projects 
WHERE
	layer = 1 
AND
IF
	( '${text}' != '', text LIKE '${text}', 1 = 1 ) 
GROUP BY
	id_
```
- sql2
```sql
SELECT
	count( * ),
	substring( zyxm, 1, 2 ) dl 
FROM
	biz_contract_stall 
WHERE
	sfzz = 'N' 
	AND end_time >= DATE( CURDATE( ) ) 
	AND YEAR ( start_time ) = YEAR ( now( ) ) 
GROUP BY
	substring( zyxm, 1, 2 );
```
#### 当前年月日业态摊位占比分析
**显示值**
- 取业态摊位占比分析表的名称和租户个数

### 业态租金占比分析
#### 当前年月日业态租金占比分析
1. **目的**
- 统计查询时间内的各经营大类的摊位费和、实收摊位费和、占总数比例
<br> *注1：实收摊位费=实收租金+实收管理费，合同摊位费=合同租金加合同管理费（不含印花税、保证金）*
<br> *注2：查询合同截至日期小于等于当前日期*
2. **筛选表**
- biz_contract_stall 合同表
- biz_contract_stall_performance 合同履行情况表
- biz_cbd_business_projects 经营品类表
3. **显示值**
- 类别编号
- 名称（取sql1里的id_字段数据）
- 合同摊位费（取sql2里合同金额数据）=各个大类的total_amount（合同总金额）字段的总和
- 实收摊位费（取sql3里的实收金额金额）=各个大类的ssglf（实收管理费）字段总和+sszj（实收租金）字段的总和
- 占总数比例=单个类别的实收摊位费/所以类别的实收摊位费*100,保留两位小数
4. **筛选条件**
- sql1：查询经营品类表为大类的品类数据
- sql2：查询合同表不是终止的合同、合同截止时间大于当前时间、合同起始时间的年份是当前年
- sql3：合同唯一编号等于履行情况的唯一编号、合同没有终止的、合同截止时间大于当前时间、合同起始时间的年份是当前年
5. **sql脚本**
- **sql1**
```sql
SELECT
	* 
FROM
	biz_cbd_business_projects 
WHERE
	layer = 1 
	AND subject_type = '大类' 
	AND text LIKE '%${type}%';
```
- **sql2**
```sql
SELECT
	sum( c.total_amount ) '合同金额',
	substring( c.zyxm, 1, 2 ) '大类ID' 
FROM
	biz_contract_stall AS c 
WHERE
	sfzz = 'N' 
	AND end_time >= DATE( CURDATE( ) ) 
	AND YEAR ( start_time ) = YEAR ( now( ) ) 
GROUP BY
	substring( c.zyxm, 1, 2 );
```
- **sql3**
```sql
SELECT
	sum( ifnull( p.ssglf, 0 ) + ifnull( p.sszj, 0 ) ) '实收金额',
	substring( c.zyxm, 1, 2 ) '大类ID' 
FROM
	biz_contract_stall_performance p,
	biz_contract_stall c 
WHERE
	c.id_ = p.contract_id 
	AND sfzz = 'N' 
	AND end_time >= DATE( CURDATE( ) ) 
	AND YEAR ( start_time ) = YEAR ( now( ) ) 
GROUP BY
	substring( c.zyxm, 1, 2 )
```

#### 当前年月日业态资金占比分析
**显示值**
- 取表格的名称和实收摊位费两列数据
### 业态坪效分析
#### 业态租金排名
1. **目的**
- 统计各经营品类的租金，并从大到小排序
2. **筛选表**
- biz_contract_stall（合同表）
- biz_cbd_business_projects（经营品类表）
3. **显示值**
- 经营大类（biz_cbd_business_projects的text）
- 各个经营大类的租金（biz_contract_stall的摊位费总和）
4. **筛选条件**
- 合同的起始时间等于当前年份
- 没有终止的合同
- 根据经营大类分组
- 根据合同的摊位费升序排序
5. **sql脚本**
```sql
SELECT
	project.text AS 经营项目,
	round( SUM( contract.twf ) / 10000, 2 ) AS 摊位费 
FROM
	biz_contract_stall AS contract -- 合同表
	LEFT JOIN biz_cbd_business_projects AS project -- 经营品类表 查询品类数据
	ON SUBSTR( contract.zyxm, '1', '2' ) = project.id_ 
WHERE
	SUBSTR( contract.start_time, 1, 4 ) = CAST( '${year}' AS DECIMAL ) 
	AND contract.sfzz <> 'Y' 
GROUP BY
	SUBSTR( contract.zyxm, '1', '2' ) -- 根据经营品类分组
	
ORDER BY
	SUM( contract.twf ) ASC -- 根据租金排序
```


#### 业态坪效排名
1. **目的**
- 统计各经营品类大类的坪效，并按从大到小排序
<br>*注释：坪效=合同总金额/经营面积/合同天数*
2. **筛选表**
- biz_contract_stall（合同表）
- biz_store_stall（摊位表）
- biz_cbd_business_projects（经营品类表）
- fr_cbd_contract_countdays（合同经营天数表）
3. **显示值**
- 经营大类（biz_cbd_business_projects的text字段）
- 各个经营大类的坪效=各个大类的总金额和/各个摊位经营面积和/合同经营天数
4. **筛选条件**
- 合同起始时间的年份是查询年份
- 合同未终止
- 根据经营大类分组
5.**sql脚本**
```sql
SELECT
	SUBSTR( contract.zyxm, '1', '2' ) 经营项目 Id,
	p.text AS 经营项目,
	SUM( days.days ) AS 合同天数,
	SUM( contract.total_amount ) 合同总金额,
	SUM( stall.jymj ) AS 经营面积,
	CONVERT (
		SUM( contract.total_amount ) / SUM( stall.jymj ) / SUM( days.days ),
		DECIMAL ( 15, 2 ) 
	) AS 坪效 
FROM
	biz_contract_stall AS contract -- 合同表
	LEFT JOIN biz_store_stall AS stall -- 摊位 取面积
	ON contract.stall_id = stall.id_
	LEFT JOIN biz_cbd_business_projects p -- 品类类型表
	ON SUBSTR( contract.zyxm, '1', '2' ) = p.id_ -- 截取获取大类编号
	LEFT JOIN fr_cbd_contract_countdays days -- 合同分租赁方式计算合同天数数据
	ON contract.id_ = days.contractId 
WHERE
	SUBSTR( contract.start_time, 1, 4 ) = CAST( '${year}' AS DECIMAL ) 
	AND contract.sfzz <> 'Y' 
GROUP BY
	SUBSTR( contract.zyxm, '1', '2' );
```


### 退市
#### 退市及其柱状图
1. **目的**
- 统计每个月份总合同数量、退市合同数量和退市率
2. **筛选表**
- fr_cbd_tuishi（退市表）
3. **显示值**
- 月份（MONTH）
- 总合同（contract_total）
- 退市合同（ts_contract_total）
- 退市率（ts_contract_tax）
4. **筛选条件**
- 无筛选条件
5. **sql脚本**
```sql
SELECT YEAR
	年,
	MONTH 月,
	contract_total 当前合同总数,
	ts_contract_total 退市合同总数,
	ts_contract_tax 退市合同率 
FROM
	fr_cbd_tuishi 
ORDER BY
MONTH
```
### 招商
#### 招商
1. **目的**
- 统计