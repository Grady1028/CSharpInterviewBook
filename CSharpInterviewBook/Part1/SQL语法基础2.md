����sql������SqlServer���ݿ�

#### ��ҳ�ļ���д����
��ѯTMS_Enterprise��ĵڶ�ҳ���ݣ�ÿҳ10������
1. **top + not in**
```
SELECT TOP 10 *
FROM dbo.TMS_Enterprise
WHERE Id NOT IN(SELECT TOP(10 * 2)Id FROM dbo.TMS_Enterprise ORDER BY Id)
ORDER BY Id
```
**ע�⣺ǰ��order by�ֶμ�������һ��**

2. **ROW_NUMBER()**
```
SELECT * 
FROM (SELECT *,ROW_NUMBER() OVER(ORDER BY Id) rown FROM dbo.TMS_Enterprise) t
WHERE t.rown BETWEEN (10 * (2-1) + 1) AND 10 * 2
```
**ע�⣺SqlServer2005����֧��**

3. **offset + fetch next ����ѣ�**
```
SELECT * FROM dbo.TMS_Enterprise
ORDER BY Id OFFSET (10 * (2-1)) ROWS FETCH NEXT 10 ROWS ONLY
```
**ע�⣺SqlServer2012����֧��**

---

#### with as �÷�
```
with
cte1 as
(
    select * from table1 where name like 'abc%'
),
cte2 as
(
    select * from table2 where id > 20
),
cte3 as
(
    select * from table3 where price < 100
)
select a.* from cte1 a, cte2 b, cte3 c where a.id = b.id and a.id = c.id
```
---

#### ʱ����غ���
- **datepart()**
```
/* datepart()������ʹ��                          
 * datepart()�������Է����ȡ��ʱ���еĸ�������
 *�����ڣ�2006-07--02 18��15��36.513
 * yy/year:ȡ��           2006
 * mm/month:ȡ��           7
 * dd/day:ȡ���е���     2
 * dy/dayofyear:ȡ���е���     183
 * wk/week:ȡ���е���     27
 * dw/weekday:ȡ���е���     1
 * qq/quarter:ȡ���еļ���   3
 * hh/hour:ȡСʱ        18
 * mi/minute:ȡ����        15
 * ss/second:ȡ��          36
 * ���¼򵥵���������ʾ��ȡ���Ľ��
 */
select getdate()
select datepart(mm,getdate())
select datepart(yy,getDate())
select datepart(dd,getdate())
select datepart(dy,getdate())
select datepart(wk,getdate())
select datepart(dw,getdate())
select datepart(qq,getdate())
select datepart(hh,getdate())
select datepart(mi,getdate())
select datepart(ss,getdate())
```
- **datediff()**
```
select datediff(dd,getdate(),'12/25/2006')����  --����ӽ��쵽12/25/2006���ж�����
select datediff(mm,getdate(),'12/25/2006')����--����ӽ��쵽12/25/2006���ж��ٸ���
```

- **dateadd()**
```
select dateadd(dd,30,getdate())           ������  --��Ŀǰ�����������ϼ�30��
select dateadd(mm,3,getdate())           ������  --��Ŀǰ�����������ϼ�3����
select dateadd(yy,1,getdate())           ��������  --��Ŀǰ�����������ϼ�1��
```
---
#### partition by �÷�

**�﷨**
```
select row_number() over(partition by A order by B ) as rowIndex from table
```
**��ṹ**
```
create table [OrderInfo](
        [Id] [int] PRIMARY KEY  IDENTITY(1,1) NOT NULL,
        [UserId] [nvarchar](50) NOT NULL,
        [TotalPrice] [float] NOT NULL,
        [OrderTime] [datetime] NOT NULL,
 );
```

1. **���ж������տͻ����з��飬�����տͻ��µĶ����Ľ������У�**
```
select Id,UserId,orderTime,ROW_NUMBER() over(partition by UserId order by TotalPrice desc) as rowIndex from OrderInfo
```

2. **ɸѡ���ͻ���һ���µĶ���**
```
 with
 baseDate
 as
 (
     select Id,UserId,TotalPrice,orderTime,ROW_NUMBER() over (partition by UserId order by orderTime) as rowIndex from OrderInfo
 )
 select * from baseDate where rowIndex=1
```