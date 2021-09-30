## 074943777a748930b2e2d19018885185 已优化

```sql
update dms_inventory_warning set is_delete = 1 where type = 1 and is_delete = 0;
```

我们写sql的时候要对要更新的数据做到心中有数,有多少数据要更新,这种sql就是要更新的数据是无法估计的.

1.取最小id,1000一次的更新

## 送货异常相关的慢sql 大需求做

### count 300秒(965aede18cc3e6c6cd53028336e4e0f4)

```sql
select count(1) from ( select dg.goods_thumb as goodsthumb, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid from dms_sub_order dso left join (select sum(delivery_amount) as total, sub_order_id from dms_sub_order_size dsos group by dsos.sub_order_id) s on s.sub_order_id = dso.sub_order_id left join dms_goods dg on dso.skc = dg.skc where order_status_value in (24, 20, 21, 22, 23) and dso.is_delete = 0 and total < dso.order_amount * 0.7 and datediff(now(), dso.delivery_time) <= 1 union select dg.goods_thumb as goodsthumb, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dso.skc = dg.skc where order_status_value in (24, 20, 21, 22, 23) and dso.is_delete = 0 and dsos.delivery_amount < dsos.order_amount * 0.7 and datediff(now(), dso.delivery_time) <= 1 ) t;
```

### 列表100多秒(40166c23670424671522d8248a5cb843)

```sql
select dg.goods_thumb as goodsthumb, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid from dms_sub_order dso left join (select sum(delivery_amount) as total, sub_order_id from dms_sub_order_size dsos group by dsos.sub_order_id) s on s.sub_order_id = dso.sub_order_id left join dms_goods dg on dso.skc = dg.skc where order_status_value in (24, 20, 21, 22, 23) and dso.is_delete = 0 and total < dso.order_amount * 0.7 and datediff(now(), dso.delivery_time) <= 1 union select dg.goods_thumb as goodsthumb, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dso.skc = dg.skc where order_status_value in (24, 20, 21, 22, 23) and dso.is_delete = 0 and dsos.delivery_amount < dsos.order_amount * 0.7 and datediff(now(), dso.delivery_time) <= 1 limit 0, 5000;
```

这边最好业务重构了,sql已然无法修改

## 100秒的更新 89dd1a78ef17beebde7bef263121997f 

使用场景:bi文件里没有给的skc的销量都要变成默认值

问题:全表更新了

优化方案:加个索引试试,索引不行就分批更新

```sql
update dms_country_sale set sales_volume = 0 where sales_volume != 0 and update_time between '2021-01-06 00:00:00' and '2021-01-07 00:00:00';
```

```sql
alter table dms_country_sale
    add index idx_update_time (update_time)
```

加了索引后优化到了50秒,但是还是有很大的空间.

还是我们先选出最小的id,然后1000个1000个一批更新

## 1.jit订单跟进 产品确认方案中

页面慢



统计慢

## 异常订单列表

数据权限试点,每个用户配置默认的维护人组,作为搜索条件,如果没有配置就给全部,防止外部用户需要查看



### 47.75秒的sql

```sql
select count(distinct dsos.sub_order_id) as ordernumber,sum(dsos.order_amount) as totalorderamount,count(distinct dso.skc) as totalskcnumber from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.is_production_completion = 1 and dso.add_time >= '2020-08-23 00:00:00' and dso.add_time <= '2020-12-23 23:59:59';
```

**扫描行数**:11452610

**耗时**:47.75秒

**场景**:jit订单统计

![image-20201223132739921](/Users/lumac/Library/Application Support/typora-user-images/image-20201223132739921.png)

## 2.rec_order 已经优化 这张表已经没有慢sql

cb03cd77605f008633e322edb0756815

110dbf65e2c8688757f1cff7bb83d6d4

dev的orderid是有索引的,但是线上是没有索引的.这种sql按理说是不应该走到线上的.只要我们能explain一下.

```sql
UPDATE dms_rec_order
SET use_flag = 1, update_uid = '系统', update_time = '2020-12-15 07:32:34.763'
WHERE order_id = 12202219;
```

```sql

```



## 3.老大难的subOrder表

### 3.1. 11.32秒的统计

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-08-24 00:00:00' and dso.add_time <= '2020-12-24 23:59:59';
```



### d9f580778825bc34094adf0810cd80d6

sql太复杂了.大数量的导出应该怎么做.

导出

```sql
select *
from (select do.id                                                              as order_id,
             ""                                                                 as suborderid,
             dos.id                                                             as sizeprimaryid,
             null                                                               as waitontheshelf,
             null                                                               as hassenttoscnum,
             0                                                                  as cancelreasonvalue,
             '1970-01-01 08:00:01'                                              as completiontime,
             '1970-01-01 08:00:01'                                              as packagetime,
             do.skc,
             dos.size,
             dos.order_amount,
             dos.delivery_amount,
             dos.receipt_amount,
             dos.storage_amount,
             dos.defective_amount,
             dos.cut_amount,
             dos.price,
             do.currency_id,
             do.add_uid,
             do.review_uid,
             date_format(do.predict_receipt_date, '%y-%m-%d %h:%i:%s')          as predict_receipt_date,
             date_format(do.demand_receipt_date, '%y-%m-%d %h:%i:%s')           as demand_receipt_date,
             date_format(do.add_time, '%y-%m-%d %h:%i:%s')                      as add_time,
             date_format(do.delivery_time, '%y-%m-%d %h:%i:%s')                 as delivery_time,
             date_format(do.receipt_time, '%y-%m-%d %h:%i:%s')                  as receipt_time,
             date_format(do.storage_time, '%y-%m-%d %h:%i:%s')                  as storage_time,
             date_format(do.refused_time, '%y-%m-%d %h:%i:%s')                  as refusedtime,
             date_format(do.offshelf_time, '%y-%m-%d %h:%i:%s')                 as offshelftime,
             date_format(do.obsolete_time, '%y-%m-%d %h:%i:%s')                 as obsoletetime,
             do.system_flag,
             do.order_status_value,
             do.supplier_id,
             do.first_mark_value,
             do.prepare_type_value,
             do.order_mark_value,
             do.urgency_value,
             do.fba_account,
             do.is_production_completion,
             do.delivery_tag,
             do.delivery_tag_uid,
             date_format(do.delivery_tag_time, '%y-%m-%d %h:%i:%s')             as delivery_tag_time,
             do.is_end_year_first_order,
             date_format(do.demand_finish_production_time, '%y-%m-%d %h:%i:%s') as demandfinishproductiontime,
             do.production_team_id                                              as productionteamid,
             do.maintenance_name                                                as maintenanceuid,
             do.category_value                                                  as categoryvalue,
             do.remark
      from dms_order do
               join dms_order_size dos on do.id = dos.order_id and dos.order_amount > 0
      where do.is_delete = 0
        and do.add_time >= '2021-01-05 00:00:00'
        and do.add_time <= '2021-01-05 23:59:59'
        and do.delivery_time >= '2021-01-18 00:00:00'
        and do.delivery_time <= '2021-01-24 23:59:59'
        and do.order_status_value in (6)
      union all
      select doo.id                                                              as order_id,
             ""                                                                  as suborderid,
             doos.id                                                             as sizeprimaryid,
             null                                                                as waitontheshelf,
             null                                                                as hassenttoscnum,
             0                                                                   as cancelreasonvalue,
             '1970-01-01 08:00:01'                                               as completiontime,
             '1970-01-01 08:00:01'                                               as packagetime,
             doo.skc,
             doos.size,
             doos.order_amount,
             doos.delivery_amount,
             doos.receipt_amount,
             doos.storage_amount,
             doos.defective_amount,
             doos.cut_amount,
             doos.price,
             doo.currency_id,
             doo.add_uid,
             doo.review_uid,
             date_format(doo.predict_receipt_date, '%y-%m-%d %h:%i:%s')          as predict_receipt_date,
             date_format(doo.demand_receipt_date, '%y-%m-%d %h:%i:%s')           as demand_receipt_date,
             date_format(doo.add_time, '%y-%m-%d %h:%i:%s')                      as add_time,
             date_format(doo.delivery_time, '%y-%m-%d %h:%i:%s')                 as delivery_time,
             date_format(doo.receipt_time, '%y-%m-%d %h:%i:%s')                  as receipt_time,
             date_format(doo.storage_time, '%y-%m-%d %h:%i:%s')                  as storage_time,
             date_format(doo.refused_time, '%y-%m-%d %h:%i:%s')                  as refusedtime,
             date_format(doo.offshelf_time, '%y-%m-%d %h:%i:%s')                 as offshelftime,
             date_format(doo.obsolete_time, '%y-%m-%d %h:%i:%s')                 as obsoletetime,
             doo.system_flag,
             doo.order_status_value,
             doo.supplier_id,
             doo.first_mark_value,
             doo.prepare_type_value,
             doo.order_mark_value,
             doo.urgency_value,
             doo.fba_account,
             doo.is_production_completion,
             doo.delivery_tag,
             doo.delivery_tag_uid,
             date_format(doo.delivery_tag_time, '%y-%m-%d %h:%i:%s')             as delivery_tag_time,
             doo.is_end_year_first_order,
             date_format(doo.demand_finish_production_time, '%y-%m-%d %h:%i:%s') as demandfinishproductiontime,
             doo.production_team_id                                              as productionteamid,
             doo.maintenance_name                                                as maintenanceuid,
             doo.category_value                                                  as categoryvalue,
             doo.remark
      from dms_order doo
               join dms_order_size doos on doo.id = doos.order_id and doos.order_amount > 0
      where doo.id in (select distinct dr.order_id
                       from dms_sub_order do
                                left join dms_order_sub_relation dr
                                          on do.sub_order_id = dr.sub_order_id and dr.is_del = 0
                       where do.is_delete = 0
                         and do.add_time >= '2021-01-05 00:00:00'
                         and do.add_time <= '2021-01-05 23:59:59'
                         and do.delivery_time >= '2021-01-18 00:00:00'
                         and do.delivery_time <= '2021-01-24 23:59:59'
                         and do.show_flag = 0
                         and do.order_status_value in
                             (7, 8, 9, 10, 11, 12, 13, 14, 15, 20, 21, 22, 23, 24, 16, 17, 18, 19))) uniontable
order by order_id desc
limit 30000,5000;
```



## 4.已优化(赵伟伟)

```sql
select distinct skc from dms_sub_order where order_status_value not in ( 18 , 19 , 24 ) and is_delete = 0 and skc > 'tee160406059' order by skc limit 1000;
```

## 5.order表 业务限制太宽松

```sql
select date_format( doo.add_time, '%y-%m-%d' ) as add_time from dms_order doo join dms_order_size doos on doo.id=doos.order_id and doos.order_amount>0 where doo.id in( select distinct dr.order_id from dms_sub_order do left join dms_order_sub_relation dr on do.sub_order_id = dr.sub_order_id and dr.is_del = 0 where do.is_delete = 0 and do.first_mark_value = '1' and do.add_time >= '2020-09-14 00:00:00' and do.add_time <= '2020-12-20 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 ) );
```

 扫描行数:17435297

耗时:14.56

使用场景:selectNewOrderAddTime,全部订单导出,三个月的订单300多万,干

## 6.

```sql
select count(1) from ( select dos.id from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.delivery_tag_time >= '2020-12-19 00:00:00' and do.delivery_tag_time <= '2020-12-19 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 , 24 ) ) t;
```

![image-20201221133154931](/Users/lumac/Library/Application Support/typora-user-images/image-20201221133154931.png)

扫描行数:15074016

耗时:8.07

## 7.业务复杂,sql更复杂.干

查询导出母单总数

```sql
select count(size_id) from ( select dos.id as size_id from dms_order do join dms_order_size dos on do.id=dos.order_id and dos.order_amount>0 where do.is_delete = 0 and do.first_mark_value = '1' and do.add_time >= '2020-09-14 00:00:00' and do.add_time <= '2020-12-21 00:00:00' and do.order_status_value in ( 6 ) union all select distinct(doos.id) size_id from dms_order doo join dms_order_size doos on doo.id=doos.order_id and doos.order_amount>0 where doo.id in( select distinct dr.order_id from dms_sub_order do left join dms_order_sub_relation dr on do.sub_order_id = dr.sub_order_id and dr.is_del = 0 where do.is_delete = 0 and do.first_mark_value = '1' and do.add_time >= '2020-09-14 00:00:00' and do.add_time <= '2020-12-21 00:00:00' and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 ) ) ) uniontable;
```

![image-20201228145850224](/Users/lumac/Library/Application Support/typora-user-images/image-20201228145850224.png)

太复杂的sql了,只能改业务了,sql不敢动

## 8.扫描太多的行数

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-08-21 00:00:00' and dso.add_time <= '2020-12-21 23:59:59';
```

扫描行数:4453473

耗时:9.55秒

场景:jit备货跟进count

只能业务上加以限制

## 9.库存统计总销量

查出数据:2047682

耗时16秒

```sql
select ifnull(sum(t1.real_time_sales_volume),0) from dms_goods_extend t1 join dms_goods t2 on t1.goods_id=t2.id where t2.bi_effect_date = date_format(now(),'%y-%m-%d') and t1.is_delete=0 and t1.`status`=1;
```

![image-20201221164401266](/Users/lumac/Library/Application Support/typora-user-images/image-20201221164401266.png)

## 10.

扫描行数:99850

耗时:5.22秒

场景:

```sql
select count(1) from dms_order do left join dms_goods dg on do.skc = dg.skc where `do`.is_delete=0 and do.add_uid = '张小艳' and `do`.prepare_type_value in ('0','11','2','3','4','7','8','9') and `do`.order_type = '1' and `do`.order_status_value in ( '2' , '3' );
```

![image-20201221173244327](/Users/lumac/Library/Application Support/typora-user-images/image-20201221173244327.png)

## 11.强制索引让单个skc出了问题

扫描行数:2678026

耗时:13.39秒,业务把修改时间去了

场景:我的请求模块

优化:不能一味的走update_time索引,业务上修改时间不能不传

```sql
select count(*) from dms_request_pdc rp force index(idx_update_time) join dms_goods g on rp.skc = g.skc where rp.task_type = 3 and rp.skc in ( 'swtop07200909495' ) and rp.is_delete = 0;
```

## 12.之前是慢sql,现在已经不慢了

```sql
select count(1) from ( select dos.id from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.delivery_tag_time >= '2020-12-21 00:00:00' and do.delivery_tag_time <= '2020-12-22 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 , 24 ) ) t;
```

## 13.商品层次自动化 

**扫描行数**:1614473

**耗时**:6.55秒

**场景**:商品层次自动化

```sql
select dglaa.* from dms_goods_level_auto_alteration dglaa left join dms_goods dg on dglaa.skc = dg.skc where dglaa.is_delete = 0 and dglaa.is_auto_generate = 1 and dglaa.target_goods_level_id = 4 and dglaa.status = 1 and dglaa.maintenance_uid in ( 133 , 96 ) and dg.goods_level_id in ( 10 ) and dg.goods_level_id in ( 4 , 10 , 12 , 25 , 43 , 44 , 45 , 46 , 47 , 51 , 52 , 55 , 61 , 62 , 66 , 67 , 68 , 70 , 75 , 77 , 79 , 80 , 81 , 83 , 84 , 85 , 86 , 87 , 88 , 89 , 90 , 91 , 92 , 93 , 94 , 95 , 96 , 97 , 99 , 100 , 101 , 102 , 103 , 104 , 105 , 106 , 107 , 108 , 109 ) order by dglaa.sales_volume desc limit 0,200;
```

**问题根源**:order by导致了速度变慢

**解决方法**

```sql
SELECT dglaa.id,dglaa.skc,dglaa.sales_volume
FROM dms_goods_level_auto_alteration dglaa
	LEFT JOIN dms_goods dg ON dglaa.skc = dg.skc
WHERE dglaa.is_delete = 0
	AND dglaa.is_auto_generate = 1
	AND dglaa.target_goods_level_id = 4
	AND dglaa.status = 1
	AND dglaa.maintenance_uid IN (133, 96)
	AND dg.goods_level_id IN (4, 10, 12, 25, 43, 44, 45, 46, 47, 51, 52, 55, 61, 62, 66, 67, 68, 70, 75, 77, 79, 80, 81, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109)
ORDER BY dglaa.sales_volume desc, dglaa.id DESC
LIMIT 0,200;
```

**how it works**

## 14.300秒的sql

使用场景:送货异常

```sql
select count(1)
from (select dso.id
      from dms_sub_order dso
               left join (select sum(delivery_amount) as total, sub_order_id
                          from dms_sub_order_size dsos
                          group by dsos.sub_order_id) s on s.sub_order_id = dso.sub_order_id
               left join dms_goods dg on dso.skc = dg.skc
      where order_status_value in (24, 20, 21, 22, 23)
        and dso.is_delete = 0
        and total < dso.order_amount * 0.7
        and datediff(now(), dso.delivery_time) <= 1
      union
      select dso.id
      from dms_sub_order dso
               left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id
               left join dms_goods dg on dso.skc = dg.skc
      where order_status_value in (24, 20, 21, 22, 23)
        and dso.is_delete = 0
        and dsos.delivery_amount < dsos.order_amount * 0.7
        and datediff(now(), dso.delivery_time) <= 1) t;
```

这边是没有统计的必要的,我们只要换一种写法,就可以不用写这么恶心的sql啦.我们在一个循环里去查询数量,如果查不到数据了,我们就退出循环,否则就查询下一批,就没有汇总的必要了.我们不能先入为主的每次都要查询总量.



## 15.非常耗时的更新

场景:库存数据任务跑完后，取备货管理页面不展示且goods表7日断货、7日断码值不为0的SKC 且滞销管理页面没有的，将其置0。

![image-20201223163347421](/Users/lumac/Library/Application Support/typora-user-images/image-20201223163347421.png)

```sql
update dms_goods set seven_day_size_short = 0, seven_day_goods_short = 0 where skc in ( select skc from ( select dg.skc from dms_goods dg left join dms_unsalable_goods dug on dg.skc = dug.skc where show_status = 1 and (seven_day_goods_short != 0 or seven_day_size_short != 0) and dug.id is null) a1);
```

我们先看看里面的查询sql

```sql
select skc from ( select dg.skc from dms_goods dg left join dms_unsalable_goods dug on dg.skc = dug.skc where show_status = 1 and (seven_day_goods_short != 0 or seven_day_size_short != 0) and dug.id is null) a1
```

## 16. 90秒的更新

```sql
update dms_country_sale set sales_volume = 0 where sales_volume != 0 and update_time between '2020-12-23 00:00:00' and '2020-12-24 00:00:00';
```

**使用场景**:bi文件里没有给的skc的销量都要变成默认值

更新的数据太多,分批更新

## 17. 37.53秒的更新

```sql
update dms_goods_extend dge,dms_goods dg set dge.sales_volume = 0, dge.deposit_sale_ratio = 0, dge.deposit_sale_ratio_total = 0, dge.s_sales_volume = 0, dge.r_sales_volume = 0, dge.platform_sales_volume = 0, dge.mmc_sales_volume = 0, dge.emc_sales_volume = 0, dge.mideast_sales_volume = 0, dge.not_mideast_sales_volume = 0, dge.outlet_sales_volume = 0, dge.stock_vigilance = 0 where dg.id = dge.goods_id and dg.is_delete = 0 and dg.sales_volume = 0 and dge.is_delete = 0 and dge.sales_volume > 0;
```

## 18. 27.36的更新

```sql
update dms_schedule_goods set is_delete = 1 where (('2020-12-24 00:00:00' > add_time or add_time > '2020-12-24 23:59:59.999') or sales_volume = 0) and is_delete = 0;
```

## 19.错误的select* 已优化

耗时:5秒

```sql
select max(id) as maxcursorkey, min(id) as mincursorkey from (select * from dms_goods where is_delete = 0 and show_status = 2 limit 157679, 157679) t;
```

![image-20201224153521608](/Users/lumac/Library/Application Support/typora-user-images/image-20201224153521608.png)

select * 会多多很多内容为什么不select id呢?

修改后时间从5秒变成700ms

## 20. 5秒的更新

```sql
update dms_unsalable_goods_extend set is_delete = 1 where bi_effect_date < '2020-12-24 00:00:00' and is_delete = 0;
```

## 21 17.64秒的count 6f1482cd134b60be6e8f27d42e11ecde

```sql
select count(t3.skc) from (select t1.skc, sum(t1.ontheway) as totalontheway from `dms_goods_on_the_way` t1 join dms_goods t2 on t1.skc = t2.skc where t1.ontheway_type in (1, 2, 3, 4) and t2.show_status = 1 and t2.goods_level_id in ( 4 , 10 , 12 , 25 , 43 , 44 , 45 , 46 , 47 , 51 , 52 , 55 , 61 , 62 , 66 , 67 , 68 , 70 , 75 , 77 , 79 , 80 , 81 , 83 , 84 , 85 , 86 , 87 , 88 , 89 , 90 , 91 , 92 , 93 , 94 , 95 , 96 , 97 , 99 , 100 , 101 , 102 , 103 , 104 , 105 , 106 , 107 , 108 , 109 ) group by t1.skc having totalontheway > 0) t3;
```

使用场景:查询有在途的skc数量

```sql

```



## 22. 13秒的count

主题:联合索引怎么加最高效

使用场景:

问题:  

1.走的这个索引 `idx_system_flage_prepare_type` (`system_flag`,`prepare_type_value`) USING BTREE,而过滤数据最多的time没有走索引

1.date()函数会让这个不走索引

```sql
select count(1) from dms_sub_order where system_flag = 1 and prepare_type_value = 9 and delivery_tag = 3 and date(delivery_tag_time) = date(now()) and delivery_tag_uid = 'root';
```

![image-20201225101126303](/Users/lumac/Library/Application Support/typora-user-images/image-20201225101126303.png)

## 23. 12.72

```sql
select count(1) from ( select dos.id from dms_order do left join dms_order_size dos on do.id=dos.order_id and dos.order_amount>0 where do.is_delete = 0 and do.prepare_type_value = '9' and do.add_time >= '2020-09-26 00:00:00' and do.add_time <= '2020-12-25 23:59:59' and do.supplier_id in (1000025) and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 16 , 17 ) ) t;
```

## 24. 50秒的查询 已解决

怪sql,为什么要这么查询

使用场景:selectInfoByOrderStatusValue

订单自动审核时游标使用的最小数据id更新任务

```sql
select * from dms_order where is_delete = 0 and id > 1 and order_status_value in ( 2 , 3 ) and order_type = 1 and prepare_type_value not in ( '12' ) order by id limit 1;
```

写sql的时候没有思考吧,一天只执行一次,50秒也就忍了,要不得,要不得!!!!

优化后的sql

```sql
select * from dms_order where id = (select min(id) from dms_order where is_delete = 0 and id > 1 and order_status_value in ( 2 , 3 ) and order_type = 1 and prepare_type_value not in ( '12' ));
```



## 26. 169.01的查询

使用场景:送货异常的查询,统计以解决 页面有点棘手

```sql
select dg.goods_thumb as goodsthumb, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid from dms_sub_order dso left join (select sum(delivery_amount) as total, sub_order_id from dms_sub_order_size dsos group by dsos.sub_order_id) s on s.sub_order_id = dso.sub_order_id left join dms_goods dg on dso.skc = dg.skc where order_status_value in (24, 20, 21, 22, 23) and dso.is_delete = 0 and total < dso.order_amount * 0.7 and datediff(now(), dso.delivery_time) <= 1 union select dg.goods_thumb as goodsthumb, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dso.skc = dg.skc where order_status_value in (24, 20, 21, 22, 23) and dso.is_delete = 0 and dsos.delivery_amount < dsos.order_amount * 0.7 and datediff(now(), dso.delivery_time) <= 1 limit 0, 5000;
```

## 27. 131秒的查询(超期订单统计)

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods_extend dge on dge.skc = dso.skc and dge.size = dsos.size where dso.is_delete = 0 and dg.is_delete = 0 and dsos.is_delete = 0 and dge.is_delete = 0 and dsos.order_amount > 0 and dso.system_flag = 2 and dso.order_status_value in ( 23 , 15 , 14 , 13 , 12 , 11 , 10 , 9 , 8 , 7 ) and dso.order_mark_value not in ( '812' , '804' , '816' , '831' , '736' ) and dg.short_in_size_status = 2 and dge.stock_b = 0 and ((dso.prepare_type_value = 0 and dso.order_level = 0) or ( dso.prepare_type_value = 9 and dso.order_level = 1));
```

## 28. 95秒的查询(超期订单)

使用场景:

超期订单查询

```sql
select * from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods_extend dge on dge.skc = dso.skc and dge.size = dsos.size where dso.is_delete = 0 and dg.is_delete = 0 and dsos.is_delete = 0 and dge.is_delete = 0 and dsos.order_amount > 0 and dso.system_flag = 2 and dso.order_status_value in ( 23 , 15 , 14 , 13 , 12 , 11 , 10 , 9 , 8 , 7 ) and dso.order_mark_value not in ( '812' , '804' , '816' , '831' , '736' ) and dg.short_in_size_status = 2 and dge.stock_b = 0 and ((dso.prepare_type_value = 0 and dso.order_level = 0) or ( dso.prepare_type_value = 9 and dso.order_level = 1)) limit 5000,5000;
```

## 29 15.35秒查询skc

```sql
select distinct skc from dms_goods_extend where is_delete = 0 and procurement_status = 2 order by skc desc limit 7000, 500;
```

## 30 Join的数据有30多万

带备货标签的搜索

```sql
select * from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id left join dms_goods_expand e on e.skc=dg.skc and e.is_delete=0 join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and dg.maintenance_uid in ( '市场成衣订单组' ) and dg.sales_volume <= '99' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 88 , 10 ) and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' ) order by dg.sales_volume desc, dg.id desc limit 0,50;
```

![image-20201228133607667](/Users/lumac/Library/Application Support/typora-user-images/image-20201228133607667.png)

## 31 不带join的也很慢 已解决 半天400多条,高频sql

```sql
SELECT dg.id
FROM dms_goods dg
	LEFT JOIN dms_supplier ds ON dg.supplier_id = ds.id
	LEFT JOIN dms_goods_expand e
	ON e.skc = dg.skc
		AND e.is_delete = 0
WHERE dg.is_delete = 0
	AND dg.show_status = 2
	AND dg.maintenance_uid IN ('生产大码组')
	AND dg.warn_inventory_status = '1'
	AND dg.goods_level_id IN (10, 61)
	AND dg.deposit_sale_ratio_total <= '4'
ORDER BY dg.sales_volume DESC, dg.id DESC
LIMIT 0, 50
```

![image-20201228141802320](/Users/lumac/Library/Application Support/typora-user-images/image-20201228141802320.png)

换成id后:

![image-20201228141845744](/Users/lumac/Library/Application Support/typora-user-images/image-20201228141845744.png)

## 32 一秒多

```sql
select count(distinct dso.id) from dms_sub_order dso join dms_exception_order deo on deo.sub_order_id = dso.sub_order_id where deo.id >= 14332433 and deo.type = 2 and deo.exception_type = 13 and deo.confirm = 2 and deo.maintenance_uid in ( '132' , '131' , '130' , '129' , '128' , '127' , '126' , '125' , '124' , '123' , '122' , '121' , '120' , '119' , '118' , '117' , '116' , '115' , '114' , '113' , '112' , '111' , '110' , '109' , '108' , '107' , '106' , '105' , '104' , '103' , '102' , '101' , '100' , '97' , '96' , '95' , '94' , '92' , '91' , '90' , '89' , '88' , '87' , '86' , '85' , '82' , '81' , '80' , '79' , '78' , '77' , '76' , '75' , '74' , '73' , '72' , '71' , '69' , '65' , '49' , '47' , '46' , '45' , '44' , '43' , '42' , '41' , '40' , '37' , '36' , '35' , '34' , '29' , '28' , '26' , '25' , '21' , '18' , '17' , '16' , '15' , '12' , '8' , '7' , '6' , '3' ) and dso.add_time >= '2020-09-29 00:00:00' and dso.show_flag = 0 and dso.is_delete = 0 and deo.is_delete = 0 and dso.sub_order_id != '';
```

## 33.26bf8fa82ec794edece54a385dcb1aa2

全都是90多秒的查询

使用场景异常订单表:queryOtherOrderList

```sql
select dso.skc as skc, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid, dsos.order_amount as orderamount, dsos.size as size, dso.add_time as addtime, dso.order_status_value as orderstatus, dge.stock_b as stockb, dge.wait_ontheshelf as waitontheshelf, dge.sales_volume as salesvolume from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods_extend dge on dge.skc = dso.skc and dge.size = dsos.size where dso.is_delete = 0 and dg.is_delete = 0 and dsos.is_delete = 0 and dge.is_delete = 0 and dsos.order_amount > 0 and dso.system_flag = 2 and dso.order_status_value in ( 23 , 15 , 14 , 13 , 12 , 11 , 10 , 9 , 8 , 7 ) and dso.order_mark_value not in ( '812' , '804' , '816' , '831' , '736' ) and dg.short_in_size_status = 2 and dge.stock_b = 0 and ((dso.prepare_type_value = 0 and dso.order_level = 0) or ( dso.prepare_type_value = 9 and dso.order_level = 1)) limit 25000,5000;
```

![image-20201228150754177](/Users/lumac/Library/Application Support/typora-user-images/image-20201228150754177.png)

dg表全表扫描了:添加索引试试

## 34.4d3053c1fdf9d426f9e89d92c43ab19e

百万级别的行扫描

耗时:29秒

使用场景:统计销量sumSaleVolume

```sql
select ifnull(sum(t1.real_time_sales_volume),0) from dms_goods_extend t1 join dms_goods t2 on t1.goods_id=t2.id where t2.bi_effect_date = date_format(now(),'%y-%m-%d') and t1.is_delete=0 and t1.`status`=1;
```

![image-20201228151421361](/Users/lumac/Library/Application Support/typora-user-images/image-20201228151421361.png)

## 35.896a5b92d44a6bbbaa619dd8dcde4d6b 已解决

耗时:27秒

使用场景:

procurement_status没有索引

```sql
select distinct skc from dms_goods_extend where is_delete = 0 and procurement_status = 2 order by skc desc limit 7000, 500;
```

添加索引

## 36.c4e78ea4aaad11240bc91ec2e5a38e51 

耗时:32秒

使用场景:子单表获取id数据

时间跨度太大,只有50条数据

```sql
select dso.id from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-08-28 00:00:00' and dso.add_time <= '2020-12-28 23:59:59' order by dso.order_id desc ,dso.id asc limit 0,50;
```

![image-20201228154531949](/Users/lumac/Library/Application Support/typora-user-images/image-20201228154531949.png)

## 37.36891fe6b913b7a611bd11ae19fb261d done

高频,零点到下午3点有500多次,优先级极高,业务高频使用

耗时:1.02-7.7秒

```sql
select dg.id from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id left join dms_goods_expand e on e.skc=dg.skc and e.is_delete=0 join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and dg.sales_volume >= '350' and dg.sales_volume <= '450' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-09-20' and s_added_time >= '2020-06-11' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1733 , 1738 , 2214 , 1732 , 1740 , 1932 , 2676 , 1862 , 2195 , 1880 , 2319 , 1866 , 2192 ) order by dg.sales_volume desc, dg.id desc limit 0,50;
```

瓶颈在标签搜索

## e7820d8ca495d5d2563a3719b43f35a4

使用场景:订单审核获取最小id

![WeChatWorkScreenshot_c536e8c6-ee1f-4068-9500-7da5fe259407](/Users/lumac/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Data/1688853404399826/Cache/Image/2020-12/WeChatWorkScreenshot_c536e8c6-ee1f-4068-9500-7da5fe259407.png)

耗时:89392ms

已经优化

```sql
select id, skc, order_number, currency_id, merchandiser, predict_receipt_date, delivery_time, receipt_time, storage_time, order_status_value, remark, remark_uid, supplier_id, first_mark_value, prepare_type_value, category_value, order_mark_value, urgency_value, fba_account, delivery_address_id, order_amount, system_flag, type, add_uid, add_time, update_uid, update_time, review_uid, review_time, is_delete, last_update_time, refused_time, offshelf_time, obsolete_time, sub_order_id, order_type, source_system, demand_receipt_date, fba_barcode_absent, source_system, demand_finish_production_time, maintenance_name, production_team_id, channel_id, is_end_year_first_order, is_production_completion, delivery_tag, batch_num from dms_order where is_delete = 0 and id > 1 and order_status_value in ( 2 , 3 ) and order_type = 1 and prepare_type_value not in ( '12' ) order by id limit 1;
```

sql换写法:

![WeChatWorkScreenshot_c3166441-57cb-4078-ac3e-618bc3ef7f2c](/Users/lumac/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Data/1688853404399826/Cache/Image/2020-12/WeChatWorkScreenshot_c3166441-57cb-4078-ac3e-618bc3ef7f2c.png)

29号后就彻底消失了

![image-20210111133443565](/Users/lumac/Library/Application Support/typora-user-images/image-20210111133443565.png)

## 62c1648f4b3b7f890b42d7b794e5b559

频率:一天一次

耗时:50秒

```sql
update dms_goods_extend dge,dms_goods dg set dge.sales_volume = 0, dge.deposit_sale_ratio = 0, dge.deposit_sale_ratio_total = 0, dge.s_sales_volume = 0, dge.r_sales_volume = 0, dge.platform_sales_volume = 0, dge.mmc_sales_volume = 0, dge.emc_sales_volume = 0, dge.mideast_sales_volume = 0, dge.not_mideast_sales_volume = 0, dge.outlet_sales_volume = 0, dge.stock_vigilance = 0 where dg.id = dge.goods_id and dg.is_delete = 0 and dg.sales_volume = 0 and dge.is_delete = 0 and dge.sales_volume > 0;
```

长耗时更新,改成批量更新,更新百万级别的数据

![image-20210111105650987](/Users/lumac/Library/Application Support/typora-user-images/image-20210111105650987.png)

## b6db717f6ba92e0d493dc1be303f6135 

频率:一天一次

耗时:50秒

使用场景:biMessageConsumeService.updateGoodsExtendDefaultValue(errorSkcList);

解决方案:业务上优化,sql没什么优化的空间了,索引效果都不是很好

```sql
update dms_schedule_goods set is_delete = 1 where (('2020-12-28 00:00:00' > add_time or add_time > '2020-12-28 23:59:59.999') or sales_volume = 0) and is_delete = 0;
```

![image-20210108133047356](/Users/lumac/Library/Application Support/typora-user-images/image-20210108133047356.png)

## 5a69f34a53a963df6244865b681b0b20

使用场景:s,r销量设置0

解决方案:业务上改方案吧,sql的条件的索引区分性能都不是太好.

```sql
update dms_goods_extend set s_sales_volume = 0, r_sales_volume = 0 where sales_volume = 0 and (s_sales_volume > 0 or r_sales_volume > 0) and is_delete = 0;
```

21.91秒更新

![image-20210108132147782](/Users/lumac/Library/Application Support/typora-user-images/image-20210108132147782.png)

## 3ff38cf5fc0a0b688d5518f15eedb50a hard

太复杂了,幸运的是频率较低

使用场景:查询导出母单总数

```sql
select count(size_id) from ( select dos.id as size_id from dms_order do join dms_order_size dos on do.id=dos.order_id and dos.order_amount>0 where do.is_delete = 0 and do.add_time >= '2020-09-29 00:00:00' and do.add_time <= '2020-12-28 23:59:59' and do.delivery_time >= '2020-12-21 00:00:00' and do.delivery_time <= '2020-12-27 23:59:59' and do.order_status_value in ( 6 ) union all select distinct(doos.id) size_id from dms_order doo join dms_order_size doos on doo.id=doos.order_id and doos.order_amount>0 where doo.id in( select distinct dr.order_id from dms_sub_order do left join dms_order_sub_relation dr on do.sub_order_id = dr.sub_order_id and dr.is_del = 0 where do.is_delete = 0 and do.add_time >= '2020-09-29 00:00:00' and do.add_time <= '2020-12-28 23:59:59' and do.delivery_time >= '2020-12-21 00:00:00' and do.delivery_time <= '2020-12-27 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 , 18 , 19 ) ) ) uniontable;
```

![image-20201228163106480](/Users/lumac/Library/Application Support/typora-user-images/image-20201228163106480.png)

## 7d20e0b9ae6f5fd4b4fcf7d01e6e6e3d hard

耗时:14秒

结果返回值:24

in了太多的数据

1.6号出现过一次

```sql
select count(1)
from dms_order do
         left join dms_goods dg on do.skc = dg.skc
where `do`.is_delete = 0
  and dg.printing > 0
  and do.maintenance_name in
      ('生产毛衣组', '生产中东组', '生产大码组', '生产维护六组', '生产维护五组', '生产维护四组', '生产维护八组', '生产premium组', '生产孕妇装', '生产维护七组', '生产维护九组',
       '生产motf组')
  and `do`.supplier_id in
      (很多)
  and `do`.prepare_type_value in ('9')
  and `do`.order_type = '1'
  and `do`.order_status_value in ('2', '3');
```

![image-20201228163228746](/Users/lumac/Library/Application Support/typora-user-images/image-20201228163228746.png)

## 0cbff9f941b50cb12380a45b4e757d70 done

已经优化date没走索引

重新优化索引结构,代码也把date去掉,传参进来

```sql
select count(1) from dms_sub_order where system_flag = 1 and prepare_type_value = 9 and delivery_tag = 3 and date(delivery_tag_time) = date(now()) and delivery_tag_uid = 'root';
```

![image-20210111141654175](/Users/lumac/Library/Application Support/typora-user-images/image-20210111141654175.png)

## 59d9024c92aed20b983fae1e219fc198 done

和上面的是一样的,上面优化了,下面自然就好了

```sql
select a.skc from ( select sum(dje.jit_available_num) as total, dje.skc from dms_jit_stock djs left join dms_jit_stock_extend dje on djs.skc = dje.skc where djs.is_delete = 0 and dje.is_delete = 0 group by dje.skc) a where total = 0;
```

## 727cb30dc795d671d832b241a7348165

13秒的更新

这个逻辑有取一张表存在一张表不存在的数据,找db看看有没有其他思路

```sql
update dms_goods set seven_day_size_short = 0, seven_day_goods_short = 0 where skc in ( select skc from ( select dg.skc from dms_goods dg left join dms_unsalable_goods dug on dg.skc = dug.skc where show_status = 1 and (seven_day_goods_short != 0 or seven_day_size_short != 0) and dug.id is null) a1);
```

## 1987e786ce98df69de837625f8559c1a

耗时:11.43秒

统计数据:113537条

count太多且连表.

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods_extend dge on dge.skc = dso.skc and dge.size = dsos.size where dso.is_delete = 0 and dg.is_delete = 0 and dsos.is_delete = 0 and dge.is_delete = 0 and dsos.order_amount > 0 and dso.system_flag = 2 and dso.order_status_value in ( 23 , 15 , 14 , 13 , 12 , 11 , 10 , 9 , 8 , 7 ) and dso.order_mark_value not in ( '812' , '804' , '816' , '831' , '736' ) and dge.warehouse_inventory_sale_rate_b > 0 and dge.warehouse_inventory_sale_rate_b <= 1 and ((dso.prepare_type_value = 0 and dso.order_level = 0) or ( dso.prepare_type_value = 9 and dso.order_level = 1));
```

![image-20201228163947052](/Users/lumac/Library/Application Support/typora-user-images/image-20201228163947052.png)

## 5432b88d85035c141e7d51728754e682

10秒的删除

```sql
delete from dms_exception_goods_goods_level where sell_status = 0;
```

分批删除吧

## 4d3053c1fdf9d426f9e89d92c43ab19e

统计实时销量

这个统计了100多万的数据,太慢了,能不能不统计

耗时:4.22-17.03秒

统计结果:3103789

db:不高频的可以先放一放

```sql
select ifnull(sum(t1.real_time_sales_volume),0) from dms_goods_extend t1 join dms_goods t2 on t1.goods_id=t2.id where t2.bi_effect_date = date_format(now(),'%y-%m-%d') and t1.is_delete=0 and t1.`status`=1;
```

![image-20201228164441260](/Users/lumac/Library/Application Support/typora-user-images/image-20201228164441260.png)



## 8ba3a3a526b7f571b2b942ca4ec06a4b

耗时:46041ms

返回:28186

使用场景

deliver_tag_time有索引吗?

添加了索引,性能到1秒多了

```sql
select count(1) from ( select dos.id from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.delivery_tag_time >= '2020-12-26 00:00:00' and do.delivery_tag_time <= '2020-12-26 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 , 24 ) ) t;
```

![image-20201228170613232](/Users/lumac/Library/Application Support/typora-user-images/image-20201228170613232.png)

sql还是可以进一步优化的

```sql
SELECT count(1)
	FROM dms_sub_order do
		LEFT JOIN dms_sub_order_size dos
		ON do.sub_order_id = dos.sub_order_id
			AND dos.order_amount > 0
	WHERE do.is_delete = 0
		AND do.delivery_tag_time >= '2020-12-27 00:00:00'
		AND do.delivery_tag_time <= '2020-12-27 23:59:59'
		AND do.show_flag = 0
		AND do.order_status_value IN (8, 20, 21, 22, 23, 24)

```

优化后的执行计划

![image-20210111142008383](/Users/lumac/Library/Application Support/typora-user-images/image-20210111142008383.png)

## 868f4c5d6f60d0a143cbcf0a991e970a

扫描了500万数据

count返回值:1007774

耗时:7872ms

大数据量的问题,sql没什么空间了

```sql
SELECT COUNT(1)
FROM dms_sub_order dso
	LEFT JOIN dms_goods dg ON dg.skc = dso.skc
	LEFT JOIN dms_supplier ds ON ds.id = dso.supplier_id
WHERE dso.is_delete = 0
	AND dso.system_flag = 2
	AND dso.prepare_type_value = '9'
	AND dso.order_level = 1
	AND dso.add_time >= '2020-08-28 00:00:00'
	AND dso.add_time <= '2020-12-28 23:59:59';
```

![image-20201228171215309](/Users/lumac/Library/Application Support/typora-user-images/image-20201228171215309.png)

## 22fc9bdac1328f67415b4a3734fbf42d

耗时:8.94

一天一次非高频

```sql
delete from dms_goods_level_auto_alteration_new where (status = 1 and is_delete = 0 and is_auto_generate = 1) or is_delete = 1;
```

拆分成两个sql更新

```sql
select count(1) from dms_goods_level_auto_alteration_new where status = 1 and is_delete = 0 and is_auto_generate = 1;

select count(1) from dms_goods_level_auto_alteration_new where  is_delete = 1;
```



## 7e8ef1a9604af095c1c155b824c6bea4

```sql
select date_format( do.add_time, '%y-%m-%d' ) as add_time from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.delivery_tag_time >= '2020-12-26 00:00:00' and do.delivery_tag_time <= '2020-12-26 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 , 24 );
```

感觉分析过了

## 14c584c8924cf50b0dfb377f50191829

使用场景:exportAllGoodsInfoCount

这个sql里supplier是多余的,看看能不能去掉呢

耗时:1秒左右,1.5号之后就没有出现了

cat_id分批count吧,10个一批是816ms

```sql
select count(1) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.category_id in ( 1889 , 1926 , 2017 , 2048 , 2220 , 1890 , 1891 , 2178 , 2223 , 1892 , 1893 , 1966 , 2224 , 2708 , 1939 , 2221 , 2222 , 2280 , 2281 , 1936 , 2324 , 2325 , 2326 , 2327 , 2328 , 1942 , 2329 , 2330 , 2331 , 2332 , 2586 , 2587 , 2668 , 1727 , 1733 , 1738 , 1779 , 2214 , 1732 , 1740 , 1871 , 1912 , 2709 , 1882 , 1922 , 1780 , 1862 , 2314 , 1880 , 2319 , 1866 , 2191 , 2192 , 2278 , 2279 , 1878 , 2174 , 2175 );
```

![image-20210111111353232](/Users/lumac/Library/Application Support/typora-user-images/image-20210111111353232.png)

## 97341e4b49f499f315998cb79d84d856

耗时:8.72秒

count返回值:803453

数据太大了,限制一下

```sql
SELECT COUNT(1)
FROM dms_goods dg
	LEFT JOIN dms_goods_extend b ON b.skc = dg.skc
	LEFT JOIN dms_supplier s ON dg.supplier_id = s.id
	LEFT JOIN dms_goods_expand e ON e.skc = dg.skc
WHERE dg.is_delete = 0
	AND dg.show_status = 2
	AND b.status = 1
	AND b.is_delete = 0
	AND dg.category_id IN (1889, 1926, 2017, 2048, 2220, 1890, 1891, 2178, 2223, 1892, 1893, 1966, 2224, 2708, 1939, 2221, 2222, 2280, 2281, 1936, 2324, 2325, 2326, 2327, 2328, 1942, 2329, 2330, 2331, 2332, 2586, 2587, 2668, 1727, 1733, 1738, 1779, 2214, 1732, 1740, 1871, 1912, 2709, 1882, 1922, 1780, 1862, 2314, 1880, 2319, 1866, 2191, 2192, 2278, 2279, 1878, 2174, 2175);
```

![image-20201228172900624](/Users/lumac/Library/Application Support/typora-user-images/image-20201228172900624.png)

## c0610363ad53fd7841b973143de4ee94

supplierId太多了,分批count吧

```sql
select count(1) from dms_order do left join dms_goods dg on do.skc = dg.skc where `do`.is_delete=0 and do.maintenance_name in ( '生产毛衣组' , '生产中东组' , '生产大码组' , '生产维护六组' , '生产维护五组' , '生产维护四组' , '生产维护八组' , '生产premium组' , '生产孕妇装' , '生产维护七组' , '生产维护九组' , '生产motf组' ) and `do`.supplier_id in ( 8192 , 1000462 , 1001485 , 29699 , 1000459 , 1001482 , 1000457 , 28679 , 1000455 , 8201 , 1000454 , 8202 , 28682 , 28683 , 28684 , 1001475 , 1000449 , 1001473 , 1000448 , 27664 , 1000478 , 1000476 , 28692 , 29717 , 2070 , 27671 , 1000471 , 7199 , 27679 , 9252 , 1001510 , 2089 , 8234 , 1000484 , 28719 , 1000509 , 1001532 , 1000504 , 1000503 , 1000500 , 27709 , 1000498 , 28736 , 28737 , 1000524 , 1001547 , 28741 , 28748 , 28749 , 1001536 , 7249 , 1000542 , 2130 , 28756 , 1000538 , 1000535 , 27736 , 1000534 , 1000533 , 1000532 , 6236 , 1000531 , 1000530 , 27750 , 29798 , 1000549 , 27758 , 27767 , 7289 , 1000563 , 28796 , 28802 , 29829 , 1000585 , 29830 , 29831 , 28811 , 29836 , 8333 , 29839 , 29840 , 29841 , 29842 , 28818 , 27805 , 27807 , 1000620 , 1000619 , 1000614 , 1000612 , 29867 , 1000611 , 27823 , 29887 , 1000654 , 7364 , 1223 , 29896 , 1000645 , 29898 , 29899 , 9420 , 9421 , 10447 , 10448 , 1000670 , 1000668 , 29910 , 7384 , 27864 , 29916 , 1000657 , 7392 , 7394 , 7395 , 1000680 , 28907 , 5361 , 1000702 , 29941 , 29942 , 1000697 , 1000695 , 1000693 , 27908 , 27909 , 1000710 , 1000706 , 27918 , 29967 , 10511 , 27920 , 27921 , 9490 , 5395 , 9491 , 1000750 , 29991 , 5416 , 5417 , 5418 , 5420 , 27948 , 9517 , 1000738 , 9519 , 1000761 , 5432 , 1000759 , 1000756 , 8508 , 1000780 , 1000777 , 7498 , 27981 , 27982 , 10574 , 5455 , 10575 , 5456 , 10576 , 1000798 , 5458 , 1000796 , 27993 , 1000787 , 1000785 , 5472 , 7522 , 7523 , 1000812 , 1000811 , 7526 , 1000809 , 1000808 , 1000807 , 7529 , 28019 , 1000828 , 1000822 , 28029 , 1000817 , 1000847 , 1000839 , 1000857 , 28058 , 1000850 , 1000848 , 30116 , 1000873 , 1000872 , 1000868 , 5550 , 5552 , 5553 , 29105 , 1000894 , 5554 , 1000888 , 1000904 , 1000903 , 1000902 , 1000901 , 1000899 , 1000927 , 1488 , 1491 , 1493 , 7640 , 1496 , 7641 , 1497 , 1498 , 1000917 , 1499 , 1000916 , 1500 , 1501 , 1000914 , 1000913 , 9694 , 1503 , 1000912 , 1000942 , 9700 , 1000935 , 1000933 , 1000930 , 1000929 , 7666 , 1528 , 1000950 , 1000947 , 5633 , 1000974 , 5634 , 1000973 , 1539 , 5635 , 5636 , 1000971 , 1000969 , 1000967 , 1000966 , 1000965 , 9742 , 9743 , 9744 , 9745 , 1000990 , 9746 , 9750 , 1000985 , 1000982 , 1562 , 1579 , 1583 , 1586 , 1001017 , 9789 , 1001010 , 8766 , 1000015 , 1000014 , 8775 , 1001032 , 1000031 , 1000030 , 1001054 , 1000029 , 1000028 , 1001052 , 1000027 , 7765 , 1000026 , 7766 , 1000025 , 7767 , 1000024 , 1000023 , 1000022 , 1000021 , 1000020 , 1001044 , 1000019 , 1000018 , 1000017 , 1000016 , 1000047 , 1000046 , 10849 , 1000045 , 1000044 , 1000043 , 1000042 , 1000041 , 1000040 , 1000039 , 1000038 , 1001061 , 30314 , 10858 , 1000037 , 1000036 , 1000035 , 1000034 , 1000033 , 1000032 , 10864 , 1000062 , 1000061 , 1000060 , 1001084 , 1000059 , 1000058 , 1000057 , 1000056 , 1000055 , 1000054 , 1001078 , 1001077 , 1000053 , 1001076 , 1000052 , 1001075 , 1000051 , 30333 , 1000049 , 1001073 , 1001072 , 1000048 , 10881 , 10882 , 10883 , 29316 , 10884 , 10885 , 1001097 , 1000069 , 10891 , 30352 , 10896 , 1001118 , 1000093 , 1000092 , 1000091 , 9877 , 1001114 , 1000090 , 1000089 , 1000088 , 1000087 , 1000086 , 1000082 , 1001105 , 30367 , 1001134 , 5796 , 5797 , 6821 , 1000105 , 29351 , 1000103 , 1000102 , 1000099 , 1001122 , 8879 , 1001120 , 1001151 , 8880 , 8881 , 1713 , 8882 , 8883 , 8884 , 1001147 , 8885 , 8886 , 1000119 , 8889 , 1001142 , 1001139 , 1727 , 6847 , 1728 , 1729 , 1001162 , 8902 , 1001156 , 1001152 , 1001178 , 8923 , 1001199 , 29417 , 1001185 , 8946 , 7927 , 28407 , 7928 , 28412 , 6910 , 1001230 , 1001225 , 1000200 , 7943 , 1000198 , 1000223 , 1000218 , 6936 , 6937 , 1001237 , 6941 , 5917 , 1001234 , 5918 , 6942 , 1000209 , 1001262 , 1000232 , 1001255 , 28457 , 1001254 , 28458 , 1837 , 1000226 , 29485 , 27438 , 9010 , 1001277 , 9011 , 9012 , 9013 , 1000249 , 1001273 , 9015 , 7991 , 29496 , 1849 , 29498 , 5953 , 5954 , 1001289 , 1000265 , 29514 , 1000259 , 28492 , 1001282 , 1000287 , 1001310 , 1001308 , 1000283 , 1000282 , 1001305 , 1000281 , 1000280 , 1001303 , 1000279 , 1000278 , 1000277 , 28507 , 1001300 , 1001299 , 1000275 , 28508 , 1000274 , 1000273 , 28513 , 8035 , 5987 , 5988 , 1001323 , 28516 , 1001322 , 1000298 , 1001321 , 1000296 , 1000295 , 1001319 , 1000294 , 1000292 , 1001315 , 1000290 , 1001314 , 28527 , 28528 , 1000318 , 28532 , 1000310 , 1000309 , 1000308 , 1000307 , 1000306 , 8062 , 1000305 , 8063 , 28543 , 1001328 , 1000304 , 1001359 , 1000334 , 1000333 , 1000332 , 1000331 , 28548 , 1000329 , 1001353 , 1000328 , 6025 , 28554 , 1000324 , 1000323 , 1001345 , 1000320 , 1001373 , 1000348 , 1000342 , 9115 , 9116 , 9117 , 7072 , 7073 , 1000365 , 10152 , 29609 , 10153 , 28586 , 1000357 , 1001380 , 28587 , 29611 , 28588 , 1000355 , 28589 , 28591 , 1000381 , 1001405 , 1000376 , 9143 , 1000375 , 28602 , 1000372 , 28604 , 1000371 , 1000368 , 28610 , 1988 , 1000395 , 1992 , 1993 , 1001414 , 28617 , 1000390 , 1994 , 1001413 , 1001412 , 29643 , 8152 , 1001431 , 1001428 , 1001427 , 9182 , 28639 , 9183 , 7138 , 1001452 , 1000422 , 1000421 , 1000420 , 10222 , 29678 , 10223 , 27633 , 1000445 , 8179 , 1001468 , 9203 , 8180 , 8181 , 1000442 , 29687 , 1000437 ) and `do`.prepare_type_value in ('9') and `do`.order_type = '1' and `do`.order_status_value in ( '2' , '3' );
```

![image-20201228173127799](/Users/lumac/Library/Application Support/typora-user-images/image-20201228173127799.png)

## 896a5b92d44a6bbbaa619dd8dcde4d6b

procurement_status没有索引加了索引已解决

```sql
select distinct skc from dms_goods_extend where is_delete = 0 and procurement_status = 2 order by skc desc limit 7000, 500;
```

![image-20210111130227839](/Users/lumac/Library/Application Support/typora-user-images/image-20210111130227839.png)

## e5bd03bb28eb98909c91322c483f85fe 

耗时:8秒

使用场景:countForRuleUnmatch

influential_id添加索引试试呢,效果不大,要改写sql了.

```sql
select count(dgm.id) from dms_goods_management dgm left join dms_goods dg on dgm.skc = dg.skc left join dms_supplier ds on dg.supplier_id = ds.id where dgm.is_delete = 0 and dgm.change_source = 3 and dgm.influential_id in (14) and ( concat(ds.type, '-', ds.category_id) not in ( '1-144' , '1-143' , '1-142' , '1-149' , '1-91' , '1-90' , '1-13' , '2-69' , '1-11' , '2-68' , '1-99' , '2-67' , '1-12' , '1-97' , '1-10' , '1-98' , '2-63' , '1-96' , '1-152' , '1-151' , '1-150' , '1-159' , '2-70' , '1-20' , '1-124' , '1-123' , '1-19' , '1-122' , '1-17' , '1-18' , '1-240' , '2-1' , '1-15' , '2-2' , '1-16' , '1-249' , '1-248' , '1-247' , '1-30' , '1-253' , '1-28' , '1-131' , '1-29' , '1-250' , '1-27' , '1-46' , '1-47' , '1-181' , '1-180' , '1-40' , '1-41' , '1-223' , '1-102' , '1-189' , '1-222' , '1-221' , '1-188' , '1-187' , '1-39' , '1-186' , '1-1' , '1-185' , '1-2' , '1-184' , '1-38' , '1-183' , '1-3' , '1-4' , '1-5' , '1-109' , '1-229' , '1-6' , '1-108' , '1-7' , '1-8' , '1-227' , '1-9' , '1-226' , '1-104' , '1-225' , '1-103' , '1-224' , '1-193' , '1-58' , '1-192' , '1-191' , '1-190' , '1-54' , '1-51' , '1-113' , '1-112' , '1-48' , '1-195' , '1-49' , '1-194' , '1-119' , '1-118' , '1-239' , '1-117' , '1-116' , '1-115' , '1-236' , '1-235' , '1-160' , '1-68' , '1-69' , '1-66' , '1-67' , '1-65' , '1-201' , '1-164' , '1-163' , '1-161' , '1-209' , '1-208' , '1-207' , '1-206' , '1-205' , '1-203' , '1-169' , '1-72' , '1-79' , '1-171' , '1-170' , '1-78' , '1-75' , '1-74' , '1-179' , '1-212' , '1-211' , '1-178' , '1-210' , '1-177' , '1-176' ) or dg.goods_level_id not in (33 ,65 ,67 ,4 ,69 ,101 ,102 ,39 ,103 ,7 ,8 ,105 ,106 ,76 ,79 ,83 ,54 ,93 ) );
```

![image-20201228174227415](/Users/lumac/Library/Application Support/typora-user-images/image-20201228174227415.png)



## ee38982ab61c7fe9a239e936aa87d3c9 给赵伟伟了

耗时:6.95秒

count:0

频率19次

last_update_time数据太多,以至于没有走这个索引,条件的区分性都不是太好

使用场景:查询需要解绑的id(促销商品终止止后把未解绑的skc解绑)

解决方案:改写业务代码,1天一天的更新,不要30天大数据更新.

```sql
select das.id from dms_activity_skc_relation das join dms_activity_promotion dap on dap.promotion_id = das.promotion_id where dap.last_update_time between '2020-11-27 00:00:03.427' and '2020-12-27 00:00:03.427' and dap.available_status = 3 and das.is_del = 0;
```

![image-20210111130439131](/Users/lumac/Library/Application Support/typora-user-images/image-20210111130439131.png)

## fa72773e31361b0862197ae82582baf2

```sql
SELECT dglaa.*
FROM dms_goods_level_auto_alteration dglaa
	LEFT JOIN dms_goods dg ON dglaa.skc = dg.skc
WHERE dglaa.is_delete = 0
	AND dglaa.is_auto_generate = 1
	AND dglaa.target_goods_level_id = 4
	AND dglaa.status = 1
	AND dglaa.maintenance_uid IN (79, 12)
	AND dg.goods_level_id IN (79)
	AND dg.goods_level_id IN (4, 10, 12, 25, 43, 44, 45, 46, 47, 51, 52, 55, 61, 62, 66, 67, 68, 70, 75, 77, 79, 80, 81, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109)
ORDER BY dglaa.sales_volume DESC
LIMIT 0, 50;
```

优化方法

```sql
SELECT dglaa.*
FROM dms_goods_level_auto_alteration dglaa
	LEFT JOIN dms_goods dg ON dglaa.skc = dg.skc
WHERE dglaa.is_delete = 0
	AND dglaa.is_auto_generate = 1
	AND dglaa.target_goods_level_id = 4
	AND dglaa.status = 1
	AND dglaa.maintenance_uid IN (79, 12)
	AND dg.goods_level_id IN (79)
	AND dg.goods_level_id IN (4, 10, 12, 25, 43, 44, 45, 46, 47, 51, 52, 55, 61, 62, 66, 67, 68, 70, 75, 77, 79, 80, 81, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109)
ORDER BY dglaa.sales_volume DESC,dg.skc desc
LIMIT 0, 50;
```

两种不同的执行计划比较:![image-20210111152634731](/Users/lumac/Library/Application Support/typora-user-images/image-20210111152634731.png)

![image-20210111152710670](/Users/lumac/Library/Application Support/typora-user-images/image-20210111152710670.png)

## 7dd1775170e292f5ee88cc135c8f033d

耗时3秒

连了很多表可是用到的并不多

改成游标取吧,id比较

使用场景:jit导出,50000一批

```sql
select dso.order_id  from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.prepare_type_value = '9' and dso.order_mark_value = '733' and dso.delivery_tag in (1,2) and dso.order_status_value in ( 15 ) order by dso.order_id desc ,dso.id asc limit 250000,50000;
```

![image-20210111152817375](/Users/lumac/Library/Application Support/typora-user-images/image-20210111152817375.png)

## 51f76f7ec1b75cb68a9d62e0a96a463d

使用场景:商品层次列表展示

```sql
select dglaa.* from dms_goods_level_auto_alteration dglaa left join dms_goods dg on dglaa.skc = dg.skc where dglaa.is_delete = 0 and dglaa.is_auto_generate = 1 and dglaa.status = 1 and dglaa.maintenance_uid in ( 133 , 96 ) and dg.goods_level_id in ( 10 ) and dg.goods_level_id in ( 4 , 10 , 12 , 25 , 43 , 44 , 45 , 46 , 47 , 51 , 52 , 55 , 61 , 62 , 66 , 67 , 68 , 70 , 75 , 77 , 79 , 80 , 81 , 83 , 84 , 85 , 86 , 87 , 88 , 89 , 90 , 91 , 92 , 93 , 94 , 95 , 96 , 97 , 99 , 100 , 101 , 102 , 103 , 104 , 105 , 106 , 107 , 108 , 109 ) or1a6741e8071028c94fd5617e37684d7cder by dglaa.sales_volume desc limit 0,100;
```

![image-20201229101100941](/Users/lumac/Library/Application Support/typora-user-images/image-20201229101100941.png)

排序加个skc,效果显著

## c4e78ea4aaad11240bc91ec2e5a38e51 

订单查询

```sql
select dso.id  from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-08-29 00:00:00' and dso.add_time <= '2020-12-29 23:59:59' order by dso.order_id desc ,dso.id asc limit 0,50;
```

![image-20201229101956267](/Users/lumac/Library/Application Support/typora-user-images/image-20201229101956267.png)

去掉orderby从6秒变成了2秒多

业务上确定下这边排序的优化点

## 51f501438e57977a6b324b64b1bb9a9c

使用场景:没有定位到,得找大家帮忙

```sql
select count(1) from ( select dos.id from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 ) ) t;
```

![image-20201229103146327](/Users/lumac/Library/Application Support/typora-user-images/image-20201229103146327.png)

## 5f366cdec177671eff0cf60e62fe7b04

索引可以优化下

维护人没有索引.验收加了维护人索引之前:耗时14秒,加了之后

```sql
select dglaa.* from dms_goods_level_auto_alteration dglaa left join dms_goods dg on dglaa.skc = dg.skc where dglaa.is_delete = 0 and dglaa.is_auto_generate = 2 and dglaa.status = 1 and dglaa.maintenance_uid in ( 107 ) and dg.goods_level_id in ( 10 ) and dg.goods_level_id in ( 4 , 10 , 12 , 25 , 43 , 44 , 45 , 46 , 47 , 51 , 52 , 55 , 61 , 62 , 66 , 67 , 68 , 70 , 75 , 77 , 79 , 80 , 81 , 83 , 84 , 85 , 86 , 87 , 88 , 89 , 90 , 91 , 92 , 93 , 94 , 95 , 96 , 97 , 99 , 100 , 101 , 102 , 103 , 104 , 105 , 106 , 107 , 108 , 109 ) and dg.s_added_time >= '1971-01-01 08:00:01' and date(dg.s_added_time) <= '2020-11-29' order by dglaa.sales_volume desc limit 0,50;
```

![image-20201229103547303](/Users/lumac/Library/Application Support/typora-user-images/image-20201229103547303.png)

## 513f6e362f31143cbf0eac00bf647280

```sql
SELECT COUNT(DISTINCT dsos.sub_order_id) AS ordernumber, SUM(dsos.order_amount) AS totalorderamount
	, COUNT(DISTINCT dso.skc) AS totalskcnumber
FROM dms_sub_order dso
	LEFT JOIN dms_sub_order_size dsos ON dsos.sub_order_id = dso.sub_order_id
	LEFT JOIN dms_goods dg ON dg.skc = dso.skc
	LEFT JOIN dms_supplier ds ON ds.id = dso.supplier_id
WHERE dso.is_delete = 0
	AND dsos.is_delete = 0
	AND dso.system_flag = 2
	AND dso.prepare_type_value = '9'
	AND dso.order_level = 1
	AND dso.add_time >= '2020-11-28 00:00:00'
	AND dso.add_time <= '2020-12-28 23:59:59'
	AND dso.demand_receipt_date >= '2020-12-27 00:00:00'
	AND dso.demand_receipt_date <= '2020-12-27 23:59:59';
```

时间跨度范围太大

![image-20201229104625537](/Users/lumac/Library/Application Support/typora-user-images/image-20201229104625537.png)

## feaf49a17f1107e63e751f2e3f2038fb

分析过了

```sql
select coalesce(sum(do.order_amount), 0) from dms_order do left join dms_goods dg on do.skc = dg.skc left join dms_goods_management dgm on dgm.skc = do.skc where `do`.is_delete=0 and dg.printing > 0 and do.maintenance_name in ( '生产毛衣组' , '生产中东组' , '生产大码组' , '生产维护六组' , '生产维护五组' , '生产维护四组' , '生产维护八组' , '生产premium组' , '生产孕妇装' , '生产维护七组' , '生产维护九组' , '生产motf组' ) and `do`.supplier_id in ( 8192 , 1000462 , 1001485 , 29699 , 1000459 , 1001482 , 1000457 , 28679 , 1000455 , 8201 , 1000454 , 8202 , 28682 , 28683 , 28684 , 1001475 , 1000449 , 1001473 , 1000448 , 27664 , 1000478 , 1000476 , 28692 , 29717 , 2070 , 27671 , 1000471 , 7199 , 27679 , 9252 , 1001510 , 2089 , 8234 , 1000484 , 28719 , 1000509 , 1001532 , 1000504 , 1000503 , 1000500 , 27709 , 1000498 , 28736 , 28737 , 1000524 , 1001547 , 28741 , 28748 , 28749 , 1001536 , 7249 , 1000542 , 2130 , 28756 , 1000538 , 1000535 , 27736 , 1000534 , 1000533 , 1000532 , 6236 , 1000531 , 1000530 , 27750 , 29798 , 1000549 , 27758 , 27767 , 7289 , 1000563 , 28796 , 28802 , 29829 , 1000585 , 29830 , 29831 , 28811 , 29836 , 8333 , 29839 , 29840 , 29841 , 29842 , 28818 , 27805 , 27807 , 1000620 , 1000619 , 1000614 , 1000612 , 29867 , 1000611 , 27823 , 29887 , 1000654 , 7364 , 1223 , 29896 , 1000645 , 29898 , 29899 , 9420 , 9421 , 10447 , 10448 , 1000670 , 1000668 , 29910 , 7384 , 27864 , 29916 , 1000657 , 7392 , 7394 , 7395 , 1000680 , 28907 , 5361 , 1000702 , 29941 , 29942 , 1000697 , 1000695 , 1000693 , 27908 , 27909 , 1000710 , 1000706 , 27918 , 29967 , 10511 , 27920 , 27921 , 9490 , 5395 , 9491 , 1000750 , 29991 , 5416 , 5417 , 5418 , 5420 , 27948 , 9517 , 1000738 , 9519 , 1000761 , 5432 , 1000759 , 1000756 , 8508 , 1000780 , 1000777 , 7498 , 27981 , 27982 , 10574 , 5455 , 10575 , 5456 , 10576 , 1000798 , 5458 , 1000796 , 27993 , 1000787 , 1000785 , 5472 , 7522 , 7523 , 1000812 , 1000811 , 7526 , 1000809 , 1000808 , 1000807 , 7529 , 28019 , 1000828 , 1000822 , 28029 , 1000817 , 1000847 , 1000839 , 1000857 , 28058 , 1000850 , 1000848 , 30116 , 1000873 , 1000872 , 1000868 , 5550 , 5552 , 5553 , 29105 , 1000894 , 5554 , 1000888 , 1000904 , 1000903 , 1000902 , 1000901 , 1000899 , 1000927 , 1488 , 1491 , 1493 , 7640 , 1496 , 7641 , 1497 , 1498 , 1000917 , 1499 , 1000916 , 1500 , 1501 , 1000914 , 1000913 , 9694 , 1503 , 1000912 , 1000942 , 9700 , 1000935 , 1000933 , 1000930 , 1000929 , 7666 , 1528 , 1000950 , 1000947 , 5633 , 1000974 , 5634 , 1000973 , 1539 , 5635 , 5636 , 1000971 , 1000969 , 1000967 , 1000966 , 1000965 , 9742 , 9743 , 9744 , 9745 , 1000990 , 9746 , 9750 , 1000985 , 1000982 , 1562 , 1579 , 1583 , 1586 , 1001017 , 9789 , 1001010 , 8766 , 1000015 , 1000014 , 8775 , 1001032 , 1000031 , 1000030 , 1001054 , 1000029 , 1000028 , 1001052 , 1000027 , 7765 , 1000026 , 7766 , 1000025 , 7767 , 1000024 , 1000023 , 1000022 , 1000021 , 1000020 , 1001044 , 1000019 , 1000018 , 1000017 , 1000016 , 1000047 , 1000046 , 10849 , 1000045 , 1000044 , 1000043 , 1000042 , 1000041 , 1000040 , 1000039 , 1000038 , 1001061 , 30314 , 10858 , 1000037 , 1000036 , 1000035 , 1000034 , 1000033 , 1000032 , 10864 , 1000062 , 1000061 , 1000060 , 1001084 , 1000059 , 1000058 , 1000057 , 1000056 , 1000055 , 1000054 , 1001078 , 1001077 , 1000053 , 1001076 , 1000052 , 1001075 , 1000051 , 30333 , 1000049 , 1001073 , 1001072 , 1000048 , 10881 , 10882 , 10883 , 29316 , 10884 , 10885 , 1001097 , 1000069 , 10891 , 30352 , 10896 , 1001118 , 1000093 , 1000092 , 1000091 , 9877 , 1001114 , 1000090 , 1000089 , 1000088 , 1000087 , 1000086 , 1000082 , 1001105 , 30367 , 1001134 , 5796 , 5797 , 6821 , 1000105 , 29351 , 1000103 , 1000102 , 1000099 , 1001122 , 8879 , 1001120 , 1001151 , 8880 , 8881 , 1713 , 8882 , 8883 , 8884 , 1001147 , 8885 , 8886 , 1000119 , 8889 , 1001142 , 1001139 , 1727 , 6847 , 1728 , 1729 , 1001162 , 8902 , 1001156 , 1001152 , 1001178 , 8923 , 1001199 , 29417 , 1001185 , 8946 , 7927 , 28407 , 7928 , 28412 , 6910 , 1001230 , 1001225 , 1000200 , 7943 , 1000198 , 1000223 , 1000218 , 6936 , 6937 , 1001237 , 6941 , 5917 , 1001234 , 5918 , 6942 , 1000209 , 1001262 , 1000232 , 1001255 , 28457 , 1001254 , 28458 , 1837 , 1000226 , 29485 , 27438 , 9010 , 1001277 , 9011 , 9012 , 9013 , 1000249 , 1001273 , 9015 , 7991 , 29496 , 1849 , 29498 , 5953 , 5954 , 1001289 , 1000265 , 29514 , 1000259 , 28492 , 1001282 , 1000287 , 1001310 , 1001308 , 1000283 , 1000282 , 1001305 , 1000281 , 1000280 , 1001303 , 1000279 , 1000278 , 1000277 , 28507 , 1001300 , 1001299 , 1000275 , 28508 , 1000274 , 1000273 , 28513 , 8035 , 5987 , 5988 , 1001323 , 28516 , 1001322 , 1000298 , 1001321 , 1000296 , 1000295 , 1001319 , 1000294 , 1000292 , 1001315 , 1000290 , 1001314 , 28527 , 28528 , 1000318 , 28532 , 1000310 , 1000309 , 1000308 , 1000307 , 1000306 , 8062 , 1000305 , 8063 , 28543 , 1001328 , 1000304 , 1001359 , 1000334 , 1000333 , 1000332 , 1000331 , 28548 , 1000329 , 1001353 , 1000328 , 6025 , 28554 , 1000324 , 1000323 , 1001345 , 1000320 , 1001373 , 1000348 , 1000342 , 9115 , 9116 , 9117 , 7072 , 7073 , 1000365 , 10152 , 29609 , 10153 , 28586 , 1000357 , 1001380 , 28587 , 29611 , 28588 , 1000355 , 28589 , 28591 , 1000381 , 1001405 , 1000376 , 9143 , 1000375 , 28602 , 1000372 , 28604 , 1000371 , 1000368 , 28610 , 1988 , 1000395 , 1992 , 1993 , 1001414 , 28617 , 1000390 , 1994 , 1001413 , 1001412 , 29643 , 8152 , 1001431 , 1001428 , 1001427 , 9182 , 28639 , 9183 , 7138 , 1001452 , 1000422 , 1000421 , 1000420 , 10222 , 29678 , 10223 , 27633 , 1000445 , 8179 , 1001468 , 9203 , 8180 , 8181 , 1000442 , 29687 , 1000437 ) and `do`.prepare_type_value in ('9') and `do`.order_type = '1' and `do`.order_status_value in ( '2' , '3' );
```

![image-20201229105016448](/Users/lumac/Library/Application Support/typora-user-images/image-20201229105016448.png)

## ae8c089682bfbf081b12d6d865c44ec2

count:15470

耗时:44293ms

时间跨度太大

```sql
select count(*) from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.maintenance_name in ( '生产维护五组' ) and dso.first_mark_value = '2' and dso.is_production_completion = 1 and dso.delivery_tag in (1,2) and dso.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 ) and dso.add_time >= '2020-08-28 00:00:00' and dso.add_time <= '2020-12-28 23:59:59';
```

![image-20210111161731071](/Users/lumac/Library/Application Support/typora-user-images/image-20210111161731071.png)

## fd3126eab0b8b94372021f0d8ed98647

耗时:6.05

使用场景:子单表统计

优化建议:时间跨度太大,业务上限制,索引也没有优化的空间了

count:16340

```sql
select count(*) from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.maintenance_name in ( '生产维护五组' ) and dso.first_mark_value = '2' and dso.is_production_completion = 1 and dso.delivery_tag in (1,2) and dso.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 ) and dso.add_time >= '2020-08-28 00:00:00' and dso.add_time <= '2020-12-28 23:59:59';
```

![image-20201229132046304](/Users/lumac/Library/Application Support/typora-user-images/image-20201229132046304.png)

## ac2eb9f235f97b8e670bfa9b2763c7a1

耗时:6.03s

使用场景:

优化建议:

count:80410

```sql
select count(*) from (select s.id as id, s.supplier_id as supplierid, s.skc as skc ,s.goods_type as goodstype, e.id as eid,e.jit_available_num as jitavailablenum, e.require_delivery_num as requiredeliverynum, e.size as size from dms_jit_stock s left join dms_jit_stock_extend e on e.jit_id = s.id left join dms_goods g on g.skc = s.skc left join dms_supplier si on si.id = g.supplier_id where s.is_delete = 0 and e.is_delete = 0 and e.jit_available_num > 0 and g.sales_volume > 0 and si.id = g.supplier_id and s.goods_type = 1 order by skc)a;
```

数据量太大,得从业务上优化了

![image-20201229132459031](/Users/lumac/Library/Application Support/typora-user-images/image-20201229132459031.png)

## ee2d053582556708a69137f1694b85b8

似曾相识,上面应该分析过了

耗时:

使用场景:

优化建议:

```sql
select coalesce(sum(do.order_amount), 0) from dms_order do left join dms_goods dg on do.skc = dg.skc left join dms_goods_management dgm on dgm.skc = do.skc where `do`.is_delete=0 and do.maintenance_name in ( '生产毛衣组' , '生产中东组' , '生产大码组' , '生产维护六组' , '生产维护五组' , '生产维护四组' , '生产维护八组' , '生产premium组' , '生产孕妇装' , '生产维护七组' , '生产维护九组' , '生产motf组' ) and `do`.supplier_id in ( 8192 , 1000462 , 1001485 , 29699 , 1000459 , 1001482 , 1000457 , 28679 , 1000455 , 8201 , 1000454 , 8202 , 28682 , 28683 , 28684 , 1001475 , 1000449 , 1001473 , 1000448 , 27664 , 1000478 , 1000476 , 28692 , 29717 , 2070 , 27671 , 1000471 , 7199 , 27679 , 9252 , 1001510 , 2089 , 8234 , 1000484 , 28719 , 1000509 , 1001532 , 1000504 , 1000503 , 1000500 , 27709 , 1000498 , 28736 , 28737 , 1000524 , 1001547 , 28741 , 28748 , 28749 , 1001536 , 7249 , 1000542 , 2130 , 28756 , 1000538 , 1000535 , 27736 , 1000534 , 1000533 , 1000532 , 6236 , 1000531 , 1000530 , 27750 , 29798 , 1000549 , 27758 , 27767 , 7289 , 1000563 , 28796 , 28802 , 29829 , 1000585 , 29830 , 29831 , 28811 , 29836 , 8333 , 29839 , 29840 , 29841 , 29842 , 28818 , 27805 , 27807 , 1000620 , 1000619 , 1000614 , 1000612 , 29867 , 1000611 , 27823 , 29887 , 1000654 , 7364 , 1223 , 29896 , 1000645 , 29898 , 29899 , 9420 , 9421 , 10447 , 10448 , 1000670 , 1000668 , 29910 , 7384 , 27864 , 29916 , 1000657 , 7392 , 7394 , 7395 , 1000680 , 28907 , 5361 , 1000702 , 29941 , 29942 , 1000697 , 1000695 , 1000693 , 27908 , 27909 , 1000710 , 1000706 , 27918 , 29967 , 10511 , 27920 , 27921 , 9490 , 5395 , 9491 , 1000750 , 29991 , 5416 , 5417 , 5418 , 5420 , 27948 , 9517 , 1000738 , 9519 , 1000761 , 5432 , 1000759 , 1000756 , 8508 , 1000780 , 1000777 , 7498 , 27981 , 27982 , 10574 , 5455 , 10575 , 5456 , 10576 , 1000798 , 5458 , 1000796 , 27993 , 1000787 , 1000785 , 5472 , 7522 , 7523 , 1000812 , 1000811 , 7526 , 1000809 , 1000808 , 1000807 , 7529 , 28019 , 1000828 , 1000822 , 28029 , 1000817 , 1000847 , 1000839 , 1000857 , 28058 , 1000850 , 1000848 , 30116 , 1000873 , 1000872 , 1000868 , 5550 , 5552 , 5553 , 29105 , 1000894 , 5554 , 1000888 , 1000904 , 1000903 , 1000902 , 1000901 , 1000899 , 1000927 , 1488 , 1491 , 1493 , 7640 , 1496 , 7641 , 1497 , 1498 , 1000917 , 1499 , 1000916 , 1500 , 1501 , 1000914 , 1000913 , 9694 , 1503 , 1000912 , 1000942 , 9700 , 1000935 , 1000933 , 1000930 , 1000929 , 7666 , 1528 , 1000950 , 1000947 , 5633 , 1000974 , 5634 , 1000973 , 1539 , 5635 , 5636 , 1000971 , 1000969 , 1000967 , 1000966 , 1000965 , 9742 , 9743 , 9744 , 9745 , 1000990 , 9746 , 9750 , 1000985 , 1000982 , 1562 , 1579 , 1583 , 1586 , 1001017 , 9789 , 1001010 , 8766 , 1000015 , 1000014 , 8775 , 1001032 , 1000031 , 1000030 , 1001054 , 1000029 , 1000028 , 1001052 , 1000027 , 7765 , 1000026 , 7766 , 1000025 , 7767 , 1000024 , 1000023 , 1000022 , 1000021 , 1000020 , 1001044 , 1000019 , 1000018 , 1000017 , 1000016 , 1000047 , 1000046 , 10849 , 1000045 , 1000044 , 1000043 , 1000042 , 1000041 , 1000040 , 1000039 , 1000038 , 1001061 , 30314 , 10858 , 1000037 , 1000036 , 1000035 , 1000034 , 1000033 , 1000032 , 10864 , 1000062 , 1000061 , 1000060 , 1001084 , 1000059 , 1000058 , 1000057 , 1000056 , 1000055 , 1000054 , 1001078 , 1001077 , 1000053 , 1001076 , 1000052 , 1001075 , 1000051 , 30333 , 1000049 , 1001073 , 1001072 , 1000048 , 10881 , 10882 , 10883 , 29316 , 10884 , 10885 , 1001097 , 1000069 , 10891 , 30352 , 10896 , 1001118 , 1000093 , 1000092 , 1000091 , 9877 , 1001114 , 1000090 , 1000089 , 1000088 , 1000087 , 1000086 , 1000082 , 1001105 , 30367 , 1001134 , 5796 , 5797 , 6821 , 1000105 , 29351 , 1000103 , 1000102 , 1000099 , 1001122 , 8879 , 1001120 , 1001151 , 8880 , 8881 , 1713 , 8882 , 8883 , 8884 , 1001147 , 8885 , 8886 , 1000119 , 8889 , 1001142 , 1001139 , 1727 , 6847 , 1728 , 1729 , 1001162 , 8902 , 1001156 , 1001152 , 1001178 , 8923 , 1001199 , 29417 , 1001185 , 8946 , 7927 , 28407 , 7928 , 28412 , 6910 , 1001230 , 1001225 , 1000200 , 7943 , 1000198 , 1000223 , 1000218 , 6936 , 6937 , 1001237 , 6941 , 5917 , 1001234 , 5918 , 6942 , 1000209 , 1001262 , 1000232 , 1001255 , 28457 , 1001254 , 28458 , 1837 , 1000226 , 29485 , 27438 , 9010 , 1001277 , 9011 , 9012 , 9013 , 1000249 , 1001273 , 9015 , 7991 , 29496 , 1849 , 29498 , 5953 , 5954 , 1001289 , 1000265 , 29514 , 1000259 , 28492 , 1001282 , 1000287 , 1001310 , 1001308 , 1000283 , 1000282 , 1001305 , 1000281 , 1000280 , 1001303 , 1000279 , 1000278 , 1000277 , 28507 , 1001300 , 1001299 , 1000275 , 28508 , 1000274 , 1000273 , 28513 , 8035 , 5987 , 5988 , 1001323 , 28516 , 1001322 , 1000298 , 1001321 , 1000296 , 1000295 , 1001319 , 1000294 , 1000292 , 1001315 , 1000290 , 1001314 , 28527 , 28528 , 1000318 , 28532 , 1000310 , 1000309 , 1000308 , 1000307 , 1000306 , 8062 , 1000305 , 8063 , 28543 , 1001328 , 1000304 , 1001359 , 1000334 , 1000333 , 1000332 , 1000331 , 28548 , 1000329 , 1001353 , 1000328 , 6025 , 28554 , 1000324 , 1000323 , 1001345 , 1000320 , 1001373 , 1000348 , 1000342 , 9115 , 9116 , 9117 , 7072 , 7073 , 1000365 , 10152 , 29609 , 10153 , 28586 , 1000357 , 1001380 , 28587 , 29611 , 28588 , 1000355 , 28589 , 28591 , 1000381 , 1001405 , 1000376 , 9143 , 1000375 , 28602 , 1000372 , 28604 , 1000371 , 1000368 , 28610 , 1988 , 1000395 , 1992 , 1993 , 1001414 , 28617 , 1000390 , 1994 , 1001413 , 1001412 , 29643 , 8152 , 1001431 , 1001428 , 1001427 , 9182 , 28639 , 9183 , 7138 , 1001452 , 1000422 , 1000421 , 1000420 , 10222 , 29678 , 10223 , 27633 , 1000445 , 8179 , 1001468 , 9203 , 8180 , 8181 , 1000442 , 29687 , 1000437 ) and `do`.prepare_type_value in ('9') and `do`.order_type = '1' and `do`.order_status_value in ( '2' , '3' );
```

![image-20201229133052205](/Users/lumac/Library/Application Support/typora-user-images/image-20201229133052205.png)

## 354199c035bc571e6552041534171a36 

耗时:is_delete没有索引

使用场景:异常订单统计

优化建议:添加索引

```sql
select ifnull(min(id),0) from dms_exception_order where is_delete=0 and type=2 and exception_type=12 and add_time >='2020-12-29';
```

![image-20201229133816277](/Users/lumac/Library/Application Support/typora-user-images/image-20201229133816277.png)

如果时间跨度太大,还是慢

92535

```sql
select count(1) from dms_exception_order where is_delete=0 and type = 2 and exception_type = 13 and add_time >='2020-10-13' 
```



## 1d7896ff5b3ed17cf9cae023bc2e521c 已解决

耗时:3.44-7.3

使用场景:

优化建议:添加索引delivery_tag_time

```sql
select count(distinct dsos.sub_order_id) as ordernumber,sum(dsos.order_amount) as totalorderamount,count(distinct dso.skc) as totalskcnumber from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-08-28 00:00:00' and dso.add_time <= '2020-12-28 23:59:59' and dso.delivery_tag_time >= '2020-12-27 00:00:00' and dso.delivery_tag_time <= '2020-12-27 23:59:59';
```

![image-20201229134851123](/Users/lumac/Library/Application Support/typora-user-images/image-20201229134851123.png)

![image-20210111163841373](/Users/lumac/Library/Application Support/typora-user-images/image-20210111163841373.png)

## 327d52eef650582431fa65025fc54abb

耗时:5021ms

使用场景:

优化建议:已分析过

count:11669

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-08-28 00:00:00' and dso.add_time <= '2020-12-28 23:59:59' and dso.demand_receipt_date >= '2020-12-27 00:00:00' and dso.demand_receipt_date <= '2020-12-27 23:59:59';
```

![image-20201229135101783](/Users/lumac/Library/Application Support/typora-user-images/image-20201229135101783.png)

## 162258d5f1bc3d229748a979104b9680

耗时:7054ms

使用场景:

优化建议:业务改造分批获取addtime

```sql
select date_format( doo.add_time, '%y-%m-%d' ) as add_time from dms_order doo join dms_order_size doos on doo.id=doos.order_id and doos.order_amount>0 where doo.id in( select distinct dr.order_id from dms_sub_order do left join dms_order_sub_relation dr on do.sub_order_id = dr.sub_order_id and dr.is_del = 0 where do.is_delete = 0 and do.add_time >= '2020-09-29 00:00:00' and do.add_time <= '2020-12-28 23:59:59' and do.delivery_time >= '2020-12-21 00:00:00' and do.delivery_time <= '2020-12-27 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 , 18 , 19 ) );
```

![image-20201229135439974](/Users/lumac/Library/Application Support/typora-user-images/image-20201229135439974.png)

## 04484cfbd54df2e3fee150d03cab3230 已解决

耗时:4.9-5.7

使用场景:

优化建议:delivertagtime可以解决

```sql
select count(distinct dsos.sub_order_id) as ordernumber,sum(dsos.order_amount) as totalorderamount,count(distinct dso.skc) as totalskcnumber from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.is_production_completion = 2 and dso.add_time >= '2020-11-19 00:00:00' and dso.add_time <= '2020-12-28 23:59:59' and dso.delivery_tag_time >= '2020-12-28 00:00:00' and dso.delivery_tag_time <= '2020-12-28 23:59:59';
```

![image-20201229135949305](/Users/lumac/Library/Application Support/typora-user-images/image-20201229135949305.png)

## 1fe4965712eacc270aac5364f7b86dbc

耗时:5167ms

使用场景:

优化建议:时间跨度太大,业务改造

count:5311

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.maintenance_name in ( '生产维护五组' ) and dso.first_mark_value = '2' and dso.is_production_completion = 1 and dso.delivery_tag in (1,2) and dso.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 ) and dso.add_time >= '2020-08-28 00:00:00' and dso.add_time <= '2020-12-28 23:59:59';
```

![image-20201229140113571](/Users/lumac/Library/Application Support/typora-user-images/image-20201229140113571.png)

## 6cf10b39402f3ac31f3cec4c2b51c3dd

耗时:2.71-5.2秒 数据有变化了吗?

dbadmin执行:33ms

使用场景:

优化建议:skc太多了

```sql
select doa.id, doa.skc, doa.size, doa.order_no, doa.goods_id, doa.renewable_warehouse, doa.stock_out_time, dg.maintenance_uid as maintenancename, dg.supplier_id, dg.goods_level_id, dge.normal_ontheway, dge.special_ontheway, dge.terminer_goods, dge.terminer_ontheway, dge.wait_ontheshelf, dge.outofstock_num, dsg.type, dsg.passage, dsg.schedule_to_shelves, dsg.warehouse_inventory from dms_order_out_of_stock_audit doa left join dms_goods dg on dg.skc = doa.skc left join dms_goods_extend dge on dge.skc = doa.skc and dge.size = doa.size left join dms_schedule_goods_extend dsg on dsg.skc = doa.skc and dsg.size = doa.size where doa.is_delete = 0 and dg.is_delete = 0 and dge.is_delete = 0 and doa.skc in ( 'smtwop07190327094' , 'smtop07190314334' , 'swvest07190516107' , 'swblouse07191022786' , 'swleg07200114773' , 'sktop07191203272' , 'swtwop07200323012' , 'sktwop07200302492' , 'mmctop-rt51012l-black' , 'swdress07191202601' , 'swpants07191230479' , 'sktwop07191230207' , 'sweatsh180103780' , 'vest171205701' , 'swskirt07190315513' , 'swtee07190418617' , 'swtop07190711245' , 'swpajama07190723874' , 'swblouse07190115574' , 'swvest07190409497' , 'sktop07190507187' , 'swdress07191231926' , 'sktwop07191230769' , 'smshorts07200218998' , 'swblouse07181204874' , 'swtop07190626683' , 'swtee07190911196' , 'swbodysui07181225588' , 'swdress07200106459' , 'swdress07191228585' , 'swlounge07191231834' , 'swdress07191218587' , 'swdress07191126998' , 'swdress07200305843' , 'swdress07190606644' , 'skdress07190723063' , 'sktop07190603000' , 'swvest07191212971' , 'swtwop07190411801' , 'skdress07181226677' , 'skblouse07190724915' , 'skblouse07190923473' , 'sktop07190529241' , 'sktwop07191230054' , 'sktwop07191209826' , 'sktop07200304403' , 'swtee07181217448' , 'swdress07191220580' , 'swvest07190430389' , 'swdress07191219454' , 'shorts180404705' , 'sktop07200227669' , 'skskirt07191231713' , 'sktwop07190730719' , 'skdress07190514222' , 'sktwop07191227182' , 'bodysuit181016751' , 'swdress07191126337' , 'sktwop07190506280' , 'smtwop07190325851' , 'swskirt07191226381' , 'swshorts07191213468' , 'smtop07191021765' , 'swtop07200106265' , 'swvest07191226156' , 'skpants07200323940' , 'sktop07200325161' , 'skblouse07191230311' , 'skblouse07191209833' , 'sktwop07191128831' , 'sktwop07190508099' , 'skblouse07191126184' , 'skpants07191230790' , 'skblouse07200104982' , 'skdress07200115939' , 'skpants07190918499' , 'skpants07190923468' , 'skpants07200102563' , 'swtee07191210718' , 'skdress07191118117' , 'smtop07190902729' , 'swdress07190710923' , 'swtwop07191012666' , 'sktop07200109090' , 'swvest07190618411' , 'swjumpsui07190704067' , 'swdress07190506902' , 'swtop07190729812' , 'sktee07200103150' , 'swvest07190614865' , 'swtwop07191231085' , 'swvest07190425595' , 'swjumpsui07191121718' , 'swpants07191129474' , 'swvest07190830796' , 'swdress07190820507' , 'skdress07200107941' , 'sknight07190529303' , 'swdress07200211102' , 'swblouse07191012525' , 'swtee07191016632' , 'sktee07200310379' , 'skpants07200106211' , 'swdress07191129273' , 'sktee07190311204' , 'skpants07200116058' , 'swpajama07190713484' , 'swtee07190718098' , 'sktee07200311720' , 'sktee07200116924' , 'skpants07191102366' , 'swtee07200423108' , 'dress170711573' , 'outer180919703' , 'outer181023749' , 'outer181023707' , 'outer180912710' , 'outer180830701' , 'outer181101745' , 'swjacket07190911213' , 'outer181029737' , 'outer180904717' , 'outer181009731' , 'outer181019738' , 'outer180920716' , 'swjacket07190724260' , 'outer180904721' , 'outer181008738' , 'outer181121702' , 'outer180830713' , 'outer181112741' , 'outermmc181102701' , 'outermmc181023701' , 'outer180921702' , 'outermmc170830701' , 'swjacket07190730105' , 'outer180925701' , 'outer181011728' , 'outer180828719' , 'outermmc180813702' , 'outer181026706' , 'outer181026703' , 'outer181009758' , 'swjacket07190827393' , 'outer181102701' , 'swjacket07190716380' , 'swjacket07190713000' , 'outer181026756' , 'swjacket07190715554' , 'outer181022701' , 'swjacket07190906209' , 'swjacket07190807271' , 'outer181105704' , 'outer181106732' , 'swjacket07190730870' , 'outer180831713' , 'swjacket07190902357' , 'swjacket07190723644' , 'outer180918718' , 'outer181126729' , 'outer180920705' , 'outer181107771' , 'outer181105723' , 'swjacket07190805418' , 'outer181030763' , 'outer181025732' , 'outer181009707' , 'outer181128751' , 'outermmc180927702' , 'outer180927719' , 'outer181024703' , 'outer181106773' , 'outer181113751' , 'swjacket07190905768' , 'swjacket07190822028' , 'swjacket07190814635' , 'swjacket07181211697' , 'swjacket07190702266' , 'outermmc180930701' , 'swjacket07190911285' , 'outermmc180917701' , 'outerwear160810702' , 'swjacket07190903843' , 'outer181105719' , 'swblazer07191113408' , 'outer181106730' , 'outer181017737' , 'outer181023746' , 'outer181106736' , 'swjacket07190819165' , 'outer180829713' , 'outer181101703' , 'outer181009756' , 'outer181119728' , 'swjacket07190722191' , 'swjacket07190816111' , 'swjacket07190916548' , 'outer180911705' , 'swblazer07190930692' , 'swjacket07181218660' , 'outer180831703' , 'outer180929703' , 'outer180930701' , 'swblazer07191125902' , 'outer180918713' , 'swjacket07190909644' , 'outer181109737' , 'swjacket07190916270' , 'outer181115701' , 'outer181102763' , 'swjacket07181206726' , 'swjacket07181211872' , 'outer181122716' , 'outer181109746' , 'outer180904703' , 'outer181024766' , 'swjacket07190820913' , 'swjacket07190717487' , 'swjacket07190822682' , 'swjacket07190806632' , 'swjacket07181206108' , 'swjacket07190722491' , 'outer180927722' , 'swjacket07190906512' , 'outer181019704' , 'swjacket07190722002' , 'outwear160802719' , 'swjacket07190718498' , 'outer181015751' , 'swjacket07190718921' , 'swdress07190606103' , 'swjacket07181207786' , 'swjacket07190819999' , 'swjacket07190729252' , 'outer181114727' , 'outer181005727' , 'outer180917722' , 'swjacket07190717018' , 'outer181120701' , 'outer181130738' , 'outer181127701' , 'outer180907712' , 'outer181109730' , 'swjacket07190718165' , 'outer181108767' , 'outer180930717' , 'outer181010729' , 'outer181026751' , 'outer181016704' , 'outer181004729' , 'outer181114733' , 'swjacket07190826413' , 'swjacket07190823142' , 'swjacket07190617884' , 'swjacket07190830820' , 'swjacket07181212293' , 'swjacket07190726395' , 'swjacket07190909046' , 'outer181019736' , 'swjacket07190812738' , 'outermmc181113701' , 'swdress07190603694' , 'outer181005753' , 'swjacket07190821238' , 'outer181018708' , 'outer181107705' , 'swjacket07190708889' , 'outer181107709' , 'swjacket07190916326' , 'swjacket07190906651' , 'outer181030702' , 'swjacket07190729459' , 'outer181116753' , 'outer180829704' , 'outer181031702' , 'outer181127762' , 'swjacket07190807508' , 'swblazer07191111662' , 'swjacket07190806661' , 'swjacket07181210840' , 'outer180928701' , 'swblazer07190904300' , 'outermmc180817702' , 'swtee07200814060' , 'swblouse07200415328' , 'swtwop07200320570' , 'blouse180227702' , 'swpants07200827617' , 'swtee07200928804' , 'swtwop07200814631' , 'sksweatsh07200805588' , 'swjumpsui07190527384' , 'swblouse07200820414' , 'swdress07200715579' , 'swtee07200916946' , 'swsweater07200612429' , 'swtwop07200227134' , 'swtee07190929382' , 'swdress07200529138' , 'swdress07201021777' , 'sktee07200821798' , 'swtee07200311952' , 'swdress07191207414' , 'swtee07190305893' , 'swleg07190702065' , 'swtop07200416284' , 'swsweater07200814219' , 'smouter07200807115' , 'swtee07190911724' , 'swsweatsh07190923166' , 'swtee07190906140' , 'swdress07200914835' , 'tee181019705' , 'swtee07200828738' , 'swblouse07200822462' , 'vest180716701' , 'skdress07190402558' , 'skdress07190422526' , 'smtwop07191230369' , 'kimono170220701' , 'swvest07191225797' , 'swpants07201007082' , 'swtop07200430900' , 'swtop07191209148' , 'swblouse07181212392' , 'swtee07200325025' , 'sktwop07200206045' , 'swtee07191210918' , 'smtop07200319664' , 'swtee07200408599' , 'blouse171220703' , 'swblouse07191121597' , 'swjumpsui07200116819' , 'swjumpsui07190426754' , 'swvest07191218634' , 'swvest07200102308' , 'sweatsh180604701' , 'swlegging07191018002' , 'smtop07190306754' , 'swvest07190527854' , 'swsweatsh07200806985' , 'smtop07200316419' , 'sktwop07200325609' , 'blouse170926712' , 'swvest07200320956' , 'swtwop07191226146' , 'swvest07191211062' , 'smtop07190325293' , 'swdress07200922244' , 'swdress07190426665' , 'sweatsh180726703' , 'smshirt07200319604' , 'dress171115716' , 'sktop07191031002' , 'swtee07200511160' , 'swtee07200408393' , 'swlegging07200303900' , 'skdress07190801777' , 'blouse160226052' , 'sktop07200227547' , 'twopiece180925702' , 'bodysuit171128703' , 'swdress07200106922' , 'sktwop07191226123' , 'sweatsh180726702' , 'swblouse07200107293' , 'smtwop07200302590' , 'smpolo07190513249' , 'skjacket07200915103' , 'swdress07201013801' , 'sktwop07191209592' , 'swtee07200227203' , 'swtee07191012580' , 'swtop07201016472' , 'swnight07190322649' , 'swtwop07200217292' , 'swtee07200116633' , 'swtee07200309532' , 'swtee07200316639' , 'swvest07190726253' , 'swtee07200324187' , 'smshirt07190621544' , 'swleg07201106922' , 'swtee07200309301' , 'dress180515711' , 'swdress07200106914' , 'swtee07191109834' , 'swtee07190315003' , 'swtee07190429936' , 'swvest07200323691' , 'blouse160719516' , 'swblouse07200818918' , 'swdress07200408080' , 'swtee07200324293' , 'swlounge07200303754' , 'swtop07200422630' , 'tee180426721' , 'dress180517728' , 'swleg07200422255' , 'skblouse07200108942' , 'tee180717705' , 'swtop07200427850' , 'sktwop07201023106' , 'swdress07200218165' , 'swtee07190319799' , 'skshorts07200116143' , 'tee181102770' , 'swblouse07200427779' , 'swblouse07190401343' , 'swvest07200411100' , 'swbodysui07200402822' , 'swnight07200924633' , 'skirt170803703' , 'swtee07190319909' , 'swtop07191221026' , 'swtee07190305963' , 'swvest07200304147' , 'swblouse07200423003' , 'swtee07190718439' , 'swskirt07200311643' , 'swvest07191226308' , 'swtee07190522401' , 'swdress07190705509' , 'swtee07190624835' , 'blouse180402703' , 'swdress07190531079' , 'swtee07190321129' , 'swvest07200422833' , 'swtee07200330041' , 'swvest07200106595' , 'tee180326705' , 'swshorts07200413032' , 'skirtmmc170918702' , 'skirt171103703' , 'swjumpsui07191212831' , 'dress181025758' , 'mmc-blt1415-blk' , 'swblouse07191220846' , 'skblouse07200302263' , 'swtee07191008297' , 'swvest07200520295' , 'swdress07201103864' , 'swvest07190724865' , 'swtee07200518674' , 'swblouse07200214539' , 'smsweatsh07200908901' , 'swshorts07190527479' , 'smtop07200429735' , 'swtwop07191231400' , 'swtwop07200408327' , 'teemmc171130701' , 'swblouse07190102410' , 'swdress07201110688' , 'swtop07190401064' , 'smpants07200409298' , 'swdress07190625182' , 'swvest07190926849' , 'swtwop07190627047' , 'swtee07200325493' , 'swtee07201024372' , 'swtee07190102577' , 'blouse171117708' , 'swvest07201119197' , 'swtwop07191010376' , 'swtee07190916556' , 'dress180226747' , 'swblouse07190430176' , 'dress171113714' , 'smtop07190709628' , 'sktwop07200401007' , 'skpolo07191228927' , 'swvest07201209018' , 'swlegging07191113613' , 'smouter07190926888' , 'sktwop07200309042' , 'swjumpsui07190611059' , 'dress180607734' , 'swtee07190528097' , 'swsweatsh07200629244' , 'swtee07200102711' , 'blouse180418713' , 'twopiece181029733' , 'dress180116742' , 'swdress07200226122' , 'swblouse07191125791' , 'swarabian07190704216' , 'dress180802716' , 'skdress07190806970' , 'swouter07201029991' , 'tee181102701' , 'swtee07181212418' , 'swdress07201114711' , 'swskirt07201023274' , 'swvest07201015123' , 'swtee07191223863' , 'swblouse07200313397' , 'swdress07190927600' , 'sktwop07200527192' , 'kimono161227703' , 'swtee07200403621' , 'swbodysui07201127982' , 'swvest07190716267' , 'swtee07200108216' , 'swdress07190311485' , 'swdress07190529349' , 'swblouse07190909993' , 'skpants07191223848' , 'dress181106761' , 'jumpsuitmmc170922701' , 'swsweatsh07200910616' , 'sktee07200302030' , 'swouter07200703379' , 'skdress07191211416' , 'swtee07190708799' , 'jumpsuit180918706' , 'jumpsuit180525701' , 'sktee07191128383' , 'swtwop07200414753' , 'swvest07201202092' , 'swvest07201202555' , 'swvest07201130913' , 'swtop07200330864' , 'swpants07201027466' , 'swvest07190402280' , 'swvest07201113659' , 'sklegging07201026391' , 'sktop07200630783' , 'swvest07191204723' , 'skjumpsui07200114532' , 'swtee07190306464' , 'swjumpsui07201104191' , 'swjacket07200905035' , 'swdress07200821416' , 'swvest07201203856' , 'swdress07191227534' , 'swjacket07201016936' , 'swtwop07190502747' , 'swtwop07201110654' , 'swdress07200904999' , 'swouter07200917876' , 'swdress07200514339' , 'sktop07200401064' , 'swouter07201014572' , 'swsweater07200914060' , 'blouse180827722' , 'swbodysui07201007943' , 'swdress07200730027' , 'swvest07190719945' , 'swdress07200427613' , 'swleg07200629093' , 'swtee07200406777' , 'sknight07200817615' , 'swsleep07200512495' , 'skpolo07200227814' , 'swsweatsh07201019660' , 'swvest07200324177' , 'swskirt07191014935' , 'swblouse07201127642' , 'skouter07200714312' , 'skskirt07200325872' , 'swtee07201103804' , 'swouter07201028127' , 'swtwop07200916994' , 'skshorts07200504613' , 'swtee07201110328' , 'swdress07200923229' , 'skdress07200826888' , 'swtwop07200929562' , 'outer180910710' , 'swpants07200710864' , 'swkimono07200317419' , 'swvest07201111550' , 'swjacket07201031785' , 'swpants07200724190' , 'swjacket07200917353' , 'smsweatsh07201013274' , 'swblouse07200526786' , 'swdress07200703079' , 'swsweatsh07201030104' , 'swouter07200905502' , 'swjumpsui07201008029' , 'skdress07200504181' , 'swsweatsh07200709263' , 'swjumpsui07190916419' , 'swjumpsui07200520320' , 'swouter07200905840' , 'swpants07200323724' , 'swarabian07200512658' , 'swpants07191211662' , 'swskirt07190521612' , 'swsweatsh07201008798' , 'swpants07200928766' , 'swdress07200115117' , 'sktwop07200725791' , 'swskirt07201026313' , 'swtee07201028793' , 'swsweatsh07201020685' , 'swblouse07201205725' , 'swdress07190730037' , 'swouter07200910822' , 'swtwop07200904453' , 'swtwop07181224200' , 'swdress07200925286' , 'swvest07200511307' , 'swdress07200515071' , 'swdress07201102350' , 'swblouse07200213533' , 'swtwop07200619567' , 'swpants07200728272' , 'sktwop07200504360' , 'swjumpsui07200302987' , 'swjacket07201012810' , 'swtee07200917133' , 'swsweatsh07200619597' , 'swtee07200917773' , 'swdress07191227010' , 'swtee07201020712' , 'skdress07200408077' , 'swnight07191030315' , 'swdress07200409224' , 'blousemmc180410701' , 'skdress07200629074' , 'swblouse07190527789' , 'sksweatsh07200902818' , 'skdress07200406041' , 'swlegging07200217826' , 'swtee07200530976' , 'skdress07200408477' , 'swjacket07200822877' , 'skpants07200706270' , 'skpants07200807084' , 'smsweatsh07201015476' , 'swtee07200422077' , 'swouter07200807046' , 'swjacket07200717573' , 'swdress07201019466' , 'swleg07201016372' , 'swtwop07200227661' , 'swskirt07200311542' , 'swshorts07201211706' , 'smtop07200416422' , 'smtop07200319542' , 'swjumpsui07201014837' , 'swdress07200917576' , 'swskirt07190814779' , 'swblouse07200724458' , 'swsweatsh07201016602' , 'swdress07200527770' , 'sktop07191231151' , 'swtee07200921397' , 'swjumpsui07200309439' , 'swouter07200925808' , 'swdress07190819126' , 'swdress07190926435' , 'swdress07200320291' , 'swdress07201016545' , 'swdress07200504354' , 'swtee07190429950' , 'sktwop07201104940' , 'swdress07200706272' , 'sktwop07201012752' , 'swdress07201105320' , 'swdress07190107041' , 'swmatern07200620447' , 'swtee07201106362' , 'sktop07200925253' , 'swvest07201111400' , 'swjumpsui07201102802' , 'swjacket07201008739' , 'swouter07200729654' , 'swbodysui07200304674' , 'swjacket07200916883' , 'swshorts07200304761' , 'swblouse07201007689' , 'swvest07200804561' , 'swtop07200820342' , 'swleg07201105186' , 'swouter07200909807' , 'swvest07201013371' , 'swsweatsh07201110687' , 'swtop07200323587' , 'swmatern07200511077' , 'swvest07200508048' , 'swjumpsui07200331895' , 'swskirt07200424461' , 'swtee07190917281' , 'swdress07190527962' , 'swtop07191114933' , 'swdress07201114798' , 'swdress07191014234' , 'swmatern07200914581' , 'swsweatsh07201012902' , 'swskirt07201110361' , 'swblouse07190812621' , 'swtee07200923965' , 'swouter07200915295' , 'swblouse07201102122' , 'swvest07201211914' , 'swskirt07200608381' , 'swvest07201113941' , 'swdress07200924494' , 'swdress07200903565' , 'swtwop07201028951' , 'swtop07200723594' , 'swskirt07190612368' , 'swtee07201104752' , 'swblouse07190404299' , 'swpants07201107100' , 'skpants07200903336' , 'swdress07200925482' , 'swblouse07200305200' , 'night180314702' , 'swshorts07200226986' , 'sktop07200507667' , 'swskirt07191008019' , 'swdress07200416750' , 'swtee07200722602' , 'swlounge07201007608' , 'swsleep07200304406' , 'swouter07201020993' , 'swskirt07200807902' , 'swtee07200803469' , 'swtee07200924246' , 'swskirt07190527832' , 'swtee07200902648' , 'swdress07200106414' , 'swtee07201103365' , 'swtee07200704441' , 'swblouse07191221268' , 'swdress07200220939' , 'swblouse07200617050' , 'swtwop07191221505' , 'swdress07191125633' , 'swdress07200602227' , 'swbodysui07181218441' , 'tee180719712' , 'swblouse07200326469' , 'swblouse07190327237' , 'swblouse07200312480' , 'swtee07200327359' , 'mmc-jt7282-yew' , 'vest180227707' , 'swvest07200109656' , 'swvest07190513601' , 'vest180411704' , 'swvest07200309670' , 'swtee07191122481' , 'swtee07200411003' , 'swblouse07191211525' , 'swtee07190827844' , 'swjumpsui07191216202' , 'swtee07200416870' , 'swdress07200402541' , 'swblouse07200522423' , 'swjacket07191112255' , 'swblouse07200423116' , 'sktwop07191109338' , 'skpants07200616487' , 'smshirt07200330593' , 'swshorts07200423076' , 'swpants07200420014' , 'blouse171109703' , 'swblouse07191121980' , 'skpants07191107828' , 'swvest07200305477' , 'swvest07200702629' , 'swtop07200620211' , 'swdress07191204268' , 'swvest07190621089' , 'swtee07200313050' , 'swskirt07200325099' , 'swjumpsui07200423885' , 'swmatern07200717243' , 'swdress07191227999' , 'swtee07190904728' , 'swbodysui07191226858' , 'swtwop07200218856' , 'swpants07201008375' , 'sktwop07200104100' , 'swtop07200504324' , 'swsweater07200929402' , 'swtee07200910411' , 'swdress07191210175' , 'swdress07191121423' , 'swdress07190910405' , 'swdress07191031784' , 'swdress07200511599' , 'smtop07190411082' , 'smtop07181205782' , 'swshorts07200416957' , 'swdress07200905846' , 'swdress07201017559' , 'smsweatsh07201009202' , 'swtwop07200903413' , 'swshorts07200331640' , 'swtop07200925622' , 'swjumpsui07200804856' , 'swdress07190606965' , 'swdress07200327895' , 'swdress07200815559' , 'swjacket07201020332' , 'swdress07200914791' , 'swshorts07191231064' , 'swdress07200903021' , 'swbodysui07190903127' , 'smpants07181225230' , 'smtwop07191219484' , 'smtwop07200116518' , 'swdress07200702917' , 'swblouse07200731127' , 'sksweater07200613863' , 'swtee07201102203' , 'swtee07200702752' , 'sktwop07200618502' , 'skblouse07191030370' , 'swpants07200521905' , 'swtee07201205058' , 'swpants07200921420' , 'skpolo07200316823' , 'swjacket07200810064' , 'swarabian07201023311' , 'sweatsh181005783' , 'swtwop07200428023' , 'twopiece171103701' , 'swjacket07190805736' , 'swblouse07200602636' , 'swtee07200413669' , 'swtwop07191023970' , 'smtop07200416824' , 'smtop07200316015' , 'swsweatsh07200810988' , 'smtop07190906975' , 'smtop07190815183' , 'smsweatsh07191024275' , 'swouter07200916420' , 'swmatern07200616263' , 'swdress07190910580' , 'sktwop07200623738' , 'skdress07191228374' , 'tee170904701' , 'swblouse07200730296' , 'swblazer07200926242' , 'swarabian07200925562' , 'sktwop07201008598' , 'jacket180930713' , 'sktwop07201007621' , 'swdress07200930171' , 'sktwop07200918838' , 'swsweatsh07201017109' , 'sktee07200602774' , 'skshirt07200504061' , 'sktee07190513078' , 'swdress07201111416' , 'swlounge07201019961' , 'tee180815709' , 'sktop07201007935' , 'swsweatsh07200805131' , 'skdress07200907775' , 'swskirt07200701806' , 'swdress07201012548' , 'pants181025702' , 'mmc-dr16837-yew' , 'smsweatsh07200622406' , 'swlegging07201110056' , 'swskirt07201201417' , 'smshirt07201015121' , 'swpants07200610661' , 'swtee07200305083' , 'swtee07200827732' , 'swvest07190718506' , 'swskirt07200413543' , 'swvest07200514909' , 'swvest07200724993' , 'swskirt07200604627' , 'swblouse07200807855' , 'swtop07200603856' , 'swshorts07200818815' , 'swshorts07200727754' , 'swshorts07200402498' , 'swsweatsh07190812606' , 'swmatern07200620270' , 'swvest07200611293' , 'swdress07200806288' , 'smouter07200915775' , 'swjumpsui07200527893' , 'swskirt07200902842' , 'swblouse07200416917' , 'swshorts07200415862' , 'swtop07200926237' , 'swtwop07200416309' , 'swblouse07201009156' , 'swouter07201004744' , 'swdress07200717977' , 'swtee07191230310' , 'swjacket07200831109' , 'swvest07201012322' , 'smtwop07201022581' , 'swtee07200921698' , 'swdress07200221226' , 'swdress07200511030' , 'swblouse07200327838' , 'swpants07200707770' , 'dress171213724' , 'swdress07200529415' , 'mmc-eq9721-red' , 'swdress07200602804' , 'swdress07191217630' , 'swdress07200602660' , 'swjacket07200805723' , 'swdress07200814895' , 'swblazer07200710516' , 'swmatern07200727702' , 'swjumpsui07190805743' , 'swdress07200514638' , 'swblouse07190717472' , 'swdress07200505223' , 'swpants07200907623' , 'swskirt07191212131' , 'swdress07200803149' , 'swshorts07200331078' , 'swpants07201009876' , 'swtwop07200915361' , 'swdress07200212458' , 'swtee07190909981' , 'blouse180226710' , 'swouter07200717594' , 'swtee07200810715' , 'swpants07181211454' , 'swtwop07181224908' , 'swtwop07200402883' , 'tee180906704' , 'swdress07190929951' , 'swtwop07200504487' , 'swpants07200205298' , 'swdress07191128076' , 'swdress07190903700' , 'swsweatsh07190722233' , 'swdress07201105067' , 'skdress07200114745' , 'swdress07200908924' , 'swdress07190712130' , 'swjumpsui07200316508' , 'swblouse07190322297' , 'swvest07200710660' , 'swtwop07200815066' , 'skdress07200930497' , 'swtee07200608609' , 'swblouse07200603148' , 'swdress07191106911' , 'swmatern07200730324' , 'swmatern07200730365' , 'swmatern07200619458' , 'swmatern07200907944' , 'swmatern07200711749' , 'swmatern07200902108' , 'swmatern07200827292' , 'swmatern07200812901' , 'swmatern07200821674' , 'swmatern07200824588' , 'swmatern07200727180' , 'swmatern07200828276' , 'swmatern07200717738' , 'skjumpsui07190730481' , 'swsweatsh07200721969' , 'swblouse07200317566' , 'swtee07200813454' , 'swdress07200525810' , 'swdress07200515596' );
```

![image-20201229140348035](/Users/lumac/Library/Application Support/typora-user-images/image-20201229140348035.png)

## 6883b27ebe0e1f248472a350adef40a8

耗时:2346714ms

使用场景:统计实时销量

优化建议:数据太多,业务改造,bi_effect_date加索引试试呢,要添加索引.代码传入bi_effect_date数据

```sql
select ifnull(sum(t1.real_time_sales_volume),0) from dms_goods_extend t1 join dms_goods t2 on t1.goods_id=t2.id where t2.bi_effect_date in (date_format(now(),'%y-%m-%d'),date_format(date_sub(now(),interval 1 day),'%y-%m-%d')) and t1.is_delete=0 and t1.`status`=1;
```

![image-20201229140616766](/Users/lumac/Library/Application Support/typora-user-images/image-20201229140616766.png)

```sql
SELECT ifnull(SUM(t1.real_time_sales_volume), 0)
FROM dms_goods_extend t1
	JOIN dms_goods t2 ON t1.goods_id = t2.id
WHERE t2.bi_effect_date IN ('21-01-11','21-01-10')
	AND t1.is_delete = 0
	AND t1.`status` = 1;
```

![image-20210111164922490](/Users/lumac/Library/Application Support/typora-user-images/image-20210111164922490.png)

## 21a0eeb18ddb9c19ebb5357ce343c413

同上

耗时:

使用场景:

优化建议:

```sql
select ifnull(sum(t1.real_time_sales_volume),0) from dms_goods_extend t1 join dms_goods t2 on t1.goods_id=t2.id where t2.bi_effect_date in (date_format(now(),'%y-%m-%d'),date_format(date_sub(now(),interval 1 day),'%y-%m-%d')) and t1.is_delete=0 and t1.`status`=1;
```



## 7055310dcf84c95c0f7ec7f4275d631b

耗时:

使用场景:

优化建议:delivery_tag_time已加索引

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.add_time >= '2020-11-19 00:00:00' and dso.add_time <= '2020-12-28 23:59:59' and dso.delivery_tag_time >= '2020-12-28 00:00:00' and dso.delivery_tag_time <= '2020-12-28 23:59:59';
```

后来就没出现过

## 67a34ae790e9812d097d8f860faf629e

耗时:6095ms

使用场景:

优化建议:业务要改造了,这块已经要炸了

count:8485

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 47 , 10 , 88 ) and year(dg.s_added_time)!='1970' and dg.maintenance_uid in ( '市场家居南通组' , '常熟一组' , '市场大码十三行组' , '十三行一组' , '市场男装十三行组' );
```

![image-20201229141310182](/Users/lumac/Library/Application Support/typora-user-images/image-20201229141310182.png)

## c9800cfcce6556e2add821ae035a6743

之前有分析过了

耗时:

使用场景:

优化建议:索引可以优化

```sql
select coalesce(sum(do.order_amount), 0) from dms_order do left join dms_goods dg on do.skc = dg.skc left join dms_goods_management dgm on dgm.skc = do.skc where `do`.is_delete=0 and `do`.supplier_id in ( 8192 , 1000462 , 1001485 , 29699 , 1000459 , 1001482 , 1000457 , 28679 , 1000455 , 8201 , 1000454 , 8202 , 28682 , 28683 , 28684 , 1001475 , 1000449 , 1001473 , 1000448 , 27664 , 1000478 , 1000476 , 28692 , 29717 , 2070 , 27671 , 1000471 , 7199 , 27679 , 9252 , 1001510 , 2089 , 8234 , 1000484 , 28719 , 1000509 , 1001532 , 1000504 , 1000503 , 1000500 , 27709 , 1000498 , 28736 , 28737 , 1000524 , 1001547 , 28741 , 28748 , 28749 , 1001536 , 7249 , 1000542 , 2130 , 28756 , 1000538 , 1000535 , 27736 , 1000534 , 1000533 , 1000532 , 6236 , 1000531 , 1000530 , 27750 , 29798 , 1000549 , 27758 , 27767 , 7289 , 1000563 , 28796 , 28802 , 29829 , 1000585 , 29830 , 29831 , 28811 , 29836 , 8333 , 29839 , 29840 , 29841 , 29842 , 28818 , 27805 , 27807 , 1000620 , 1000619 , 1000614 , 1000612 , 29867 , 1000611 , 27823 , 29887 , 1000654 , 7364 , 1223 , 29896 , 1000645 , 29898 , 29899 , 9420 , 9421 , 10447 , 10448 , 1000670 , 1000668 , 29910 , 7384 , 27864 , 29916 , 1000657 , 7392 , 7394 , 7395 , 1000680 , 28907 , 5361 , 1000702 , 29941 , 29942 , 1000697 , 1000695 , 1000693 , 27908 , 27909 , 1000710 , 1000706 , 27918 , 29967 , 10511 , 27920 , 27921 , 9490 , 5395 , 9491 , 1000750 , 29991 , 5416 , 5417 , 5418 , 5420 , 27948 , 9517 , 1000738 , 9519 , 1000761 , 5432 , 1000759 , 1000756 , 8508 , 1000780 , 1000777 , 7498 , 27981 , 27982 , 10574 , 5455 , 10575 , 5456 , 10576 , 1000798 , 5458 , 1000796 , 27993 , 1000787 , 1000785 , 5472 , 7522 , 7523 , 1000812 , 1000811 , 7526 , 1000809 , 1000808 , 1000807 , 7529 , 28019 , 1000828 , 1000822 , 28029 , 1000817 , 1000847 , 1000839 , 1000857 , 28058 , 1000850 , 1000848 , 30116 , 1000873 , 1000872 , 1000868 , 5550 , 5552 , 5553 , 29105 , 1000894 , 5554 , 1000888 , 1000904 , 1000903 , 1000902 , 1000901 , 1000899 , 1000927 , 1488 , 1491 , 1493 , 7640 , 1496 , 7641 , 1497 , 1498 , 1000917 , 1499 , 1000916 , 1500 , 1501 , 1000914 , 1000913 , 9694 , 1503 , 1000912 , 1000942 , 9700 , 1000935 , 1000933 , 1000930 , 1000929 , 7666 , 1528 , 1000950 , 1000947 , 5633 , 1000974 , 5634 , 1000973 , 1539 , 5635 , 5636 , 1000971 , 1000969 , 1000967 , 1000966 , 1000965 , 9742 , 9743 , 9744 , 9745 , 1000990 , 9746 , 9750 , 1000985 , 1000982 , 1562 , 1579 , 1583 , 1586 , 1001017 , 9789 , 1001010 , 8766 , 1000015 , 1000014 , 8775 , 1001032 , 1000031 , 1000030 , 1001054 , 1000029 , 1000028 , 1001052 , 1000027 , 7765 , 1000026 , 7766 , 1000025 , 7767 , 1000024 , 1000023 , 1000022 , 1000021 , 1000020 , 1001044 , 1000019 , 1000018 , 1000017 , 1000016 , 1000047 , 1000046 , 10849 , 1000045 , 1000044 , 1000043 , 1000042 , 1000041 , 1000040 , 1000039 , 1000038 , 1001061 , 30314 , 10858 , 1000037 , 1000036 , 1000035 , 1000034 , 1000033 , 1000032 , 10864 , 1000062 , 1000061 , 1000060 , 1001084 , 1000059 , 1000058 , 1000057 , 1000056 , 1000055 , 1000054 , 1001078 , 1001077 , 1000053 , 1001076 , 1000052 , 1001075 , 1000051 , 30333 , 1000049 , 1001073 , 1001072 , 1000048 , 10881 , 10882 , 10883 , 29316 , 10884 , 10885 , 1001097 , 1000069 , 10891 , 30352 , 10896 , 1001118 , 1000093 , 1000092 , 1000091 , 9877 , 1001114 , 1000090 , 1000089 , 1000088 , 1000087 , 1000086 , 1000082 , 1001105 , 30367 , 1001134 , 5796 , 5797 , 6821 , 1000105 , 29351 , 1000103 , 1000102 , 1000099 , 1001122 , 8879 , 1001120 , 1001151 , 8880 , 8881 , 1713 , 8882 , 8883 , 8884 , 1001147 , 8885 , 8886 , 1000119 , 8889 , 1001142 , 1001139 , 1727 , 6847 , 1728 , 1729 , 1001162 , 8902 , 1001156 , 1001152 , 1001178 , 8923 , 1001199 , 29417 , 1001185 , 8946 , 7927 , 28407 , 7928 , 28412 , 6910 , 1001230 , 1001225 , 1000200 , 7943 , 1000198 , 1000223 , 1000218 , 6936 , 6937 , 1001237 , 6941 , 5917 , 1001234 , 5918 , 6942 , 1000209 , 1001262 , 1000232 , 1001255 , 28457 , 1001254 , 28458 , 1837 , 1000226 , 29485 , 27438 , 9010 , 1001277 , 9011 , 9012 , 9013 , 1000249 , 1001273 , 9015 , 7991 , 29496 , 1849 , 29498 , 5953 , 5954 , 1001289 , 1000265 , 29514 , 1000259 , 28492 , 1001282 , 1000287 , 1001310 , 1001308 , 1000283 , 1000282 , 1001305 , 1000281 , 1000280 , 1001303 , 1000279 , 1000278 , 1000277 , 28507 , 1001300 , 1001299 , 1000275 , 28508 , 1000274 , 1000273 , 28513 , 8035 , 5987 , 5988 , 1001323 , 28516 , 1001322 , 1000298 , 1001321 , 1000296 , 1000295 , 1001319 , 1000294 , 1000292 , 1001315 , 1000290 , 1001314 , 28527 , 28528 , 1000318 , 28532 , 1000310 , 1000309 , 1000308 , 1000307 , 1000306 , 8062 , 1000305 , 8063 , 28543 , 1001328 , 1000304 , 1001359 , 1000334 , 1000333 , 1000332 , 1000331 , 28548 , 1000329 , 1001353 , 1000328 , 6025 , 28554 , 1000324 , 1000323 , 1001345 , 1000320 , 1001373 , 1000348 , 1000342 , 9115 , 9116 , 9117 , 7072 , 7073 , 1000365 , 10152 , 29609 , 10153 , 28586 , 1000357 , 1001380 , 28587 , 29611 , 28588 , 1000355 , 28589 , 28591 , 1000381 , 1001405 , 1000376 , 9143 , 1000375 , 28602 , 1000372 , 28604 , 1000371 , 1000368 , 28610 , 1988 , 1000395 , 1992 , 1993 , 1001414 , 28617 , 1000390 , 1994 , 1001413 , 1001412 , 29643 , 8152 , 1001431 , 1001428 , 1001427 , 9182 , 28639 , 9183 , 7138 , 1001452 , 1000422 , 1000421 , 1000420 , 10222 , 29678 , 10223 , 27633 , 1000445 , 8179 , 1001468 , 9203 , 8180 , 8181 , 1000442 , 29687 , 1000437 ) and `do`.prepare_type_value in ('1') and `do`.order_type = '1' and `do`.order_status_value in ( '2' , '3' );
```



## 2c401e98b8dd49e7b7d538f57cc3352e

耗时:1.3-22秒

使用场景:orderCommonCondition

优化建议:分批统计吧,结果太多了,扫描1037666行

```sql
select count(1) from dms_order do left join dms_goods dg on do.skc = dg.skc where `do`.is_delete=0 and `do`.supplier_id in (太多);
```



## 1f963f292082038c7ac16a090e8ff646 已解决

耗时:

使用场景:

优化建议:已加优化

```sql
select count(1) from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.is_production_completion = 2 and dso.add_time >= '2020-11-19 00:00:00' and dso.add_time <= '2020-12-28 23:59:59' and dso.delivery_tag_time >= '2020-12-28 00:00:00' and dso.delivery_tag_time <= '2020-12-28 23:59:59';
```



## 36891fe6b913b7a611bd11ae19fb261d

耗时:2642ms

使用场景:

优化建议:

业务优化吧,备货标签搜索太耗时

```sql
select dg.id, dg.product_time, show_status, dg.skc, s_sale_status, r_sale_status, s_sale_by_inventory, r_sale_by_inventory, s_added_time, r_added_time, s_gross_profit_margin, r_gross_profit_margin, goods_thumb, average_return_time, goods_level_id, dg.maintenance_uid, short_in_skc_status, short_in_size_status, warn_inventory_status, delivery_time, supplier_id, e.buyer, supplier_code, production_team_id, production_team_name, season_id, outofstock_total, first_order_date, stock_date, bi_effect_date, dg.add_time, dg.update_time, dg.add_uid, dg.update_uid, dg.is_delete, dg.last_update_time, dg.first_order_id, order_count, deposit_sale_ratio, sales_volume, s_sales_volume, r_sales_volume, return_rate_30_days, return_rate_60_days, s_added_time, r_added_time, dg.real_time_sales_volume, e.target_inventory, e.demand_inventory, dg.eval_score, dg.material_status, ds.type, dg.printing from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id left join dms_goods_expand e on e.skc=dg.skc and e.is_delete=0 join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and dg.sales_volume >= '300' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-09-21' and s_added_time >= '2020-06-12' and dg.goods_level_id in ( 107 , 87 , 61 , 10 ) and dg.return_rate_30_days <= '8' and dg.return_rate_60_days <= '10' order by dg.sales_volume desc, dg.id desc limit 450,50;
```

![image-20201229141917275](/Users/lumac/Library/Application Support/typora-user-images/image-20201229141917275.png)

## 99a4fae60b62967356e061348fa748fd

耗时:4405ms

使用场景:

优化建议:同上

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 10 , 61 ) and dg.maintenance_uid in ( '市场中东成衣组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '流花男装组' , '样衣' );
```

![image-20201229142140036](/Users/lumac/Library/Application Support/typora-user-images/image-20201229142140036.png)

## bc391f77cfbb515c8092550e17311659

耗时:

使用场景:

优化建议:跟上面的相同的原因,业务改造吧.

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 10 , 61 ) and year(dg.s_added_time)!='1970' and dg.maintenance_uid in ( '市场中东成衣组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '流花男装组' , '样衣' );
```



## a25664ebcbff5e98fa15f897040816d1

耗时:16785ms

使用场景:

优化建议:数据量太大,这种写法并不是很好.

```sql
select dge.id, dge.skc, dge.size, dge.stock_b as inventory from dms_goods_extend dge straight_join dms_goods dg on dg.skc = dge.skc where dg.is_delete = 0 and dg.show_status = 2 and dge.is_delete = 0 and dge.status = 1 and dge.stock_b > 0 and dge.id > 0 order by dge.id limit 5000;
```

![image-20201229142600828](/Users/lumac/Library/Application Support/typora-user-images/image-20201229142600828.png)

## 0bf99096c3b52b18f3c4e20a25a3a79d

耗时:5105ms

使用场景:

优化建议:业务限制

| ordernumber | totalorderamount | totalskcnumber |      |
| :---------- | :--------------- | :------------- | :--- |
| 82298       | 7384240          | 14982          |      |

```sql
select count(distinct dsos.sub_order_id) as ordernumber,sum(dsos.order_amount) as totalorderamount,count(distinct dso.skc) as totalskcnumber from dms_sub_order dso left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods dg on dg.skc = dso.skc left join dms_supplier ds on ds.id = dso.supplier_id where dso.is_delete = 0 and dsos.is_delete = 0 and dso.system_flag = 2 and dso.prepare_type_value = '9' and dso.order_level=1 and dso.delivery_tag in (1,2) and dso.order_status_value in ( 15 );
```

![image-20201229142908789](/Users/lumac/Library/Application Support/typora-user-images/image-20201229142908789.png)

## 24d933d22b659a6e96da7a44db881869

耗时:

使用场景:

优化建议:同上面的skc备货标签

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( '沙河内衣组' , '沙河一组' ) and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-22' and dg.goods_level_id in ( 61 , 10 );
```



## cf6cfe3172ce562f38b7f4b6507f180e

耗时:

使用场景:

优化建议:同上

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( '沙河内衣组' , '沙河一组' ) and dg.sales_volume <= '99' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-22' and dg.goods_level_id in ( 61 , 10 ) and year(dg.s_added_time)!='1970';
```



## 048e7951bc487fce49678fd3309773b0

耗时:

使用场景:

优化建议:同上

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 36 and label_type_status = 2 and label_status = 2 and label_id in ( 37 ) ) r36 on dg.skc = r36.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( '市场成衣订单组' , 'al数码组' , 'al订单组' ) and dg.deposit_sale_ratio_total <= '6' and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## ff52756269d87b257236f97c300fbcd3

耗时:

使用场景:

优化建议:

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 36 and label_type_status = 2 and label_status = 2 and label_id in ( 37 ) ) r36 on dg.skc = r36.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( '市场成衣订单组' , 'al数码组' , 'al订单组' ) and dg.deposit_sale_ratio_total <= '6' and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## 212d73e95282855fd418bd5723f34062 

耗时:

使用场景:

优化建议:

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 36 and label_type_status = 2 and label_status = 2 and label_id in ( 37 ) ) r36 on dg.skc = r36.skc where dg.show_status = 2 and dg.maintenance_uid in ( '市场成衣订单组' , 'al数码组' , 'al订单组' ) and dg.deposit_sale_ratio_total <= '6' and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## c142d2fb2e7cb0105301e68317313eaf

耗时:

使用场景:

优化建议:sql太复杂

```sql
select count(size_id) from ( select dos.id as size_id from dms_order do join dms_order_size dos on do.id=dos.order_id and dos.order_amount>0 where do.is_delete = 0 and do.first_mark_value = '1' and do.add_time >= '2020-12-21 00:00:00' and do.add_time <= '2020-12-28 23:59:59' and do.order_status_value in ( 4 , 6 ) union all select distinct(doos.id) size_id from dms_order doo join dms_order_size doos on doo.id=doos.order_id and doos.order_amount>0 where doo.id in( select distinct dr.order_id from dms_sub_order do left join dms_order_sub_relation dr on do.sub_order_id = dr.sub_order_id and dr.is_del = 0 where do.is_delete = 0 and do.first_mark_value = '1' and do.add_time >= '2020-12-21 00:00:00' and do.add_time <= '2020-12-28 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 16 , 17 , 18 , 19 , 20 , 21 , 22 , 23 , 24 ) ) ) uniontable;
```

![image-20210114110723945](/Users/lumac/Library/Application Support/typora-user-images/image-20210114110723945.png)

## b99a85a83408491b9013d51e717e5197

耗时:

使用场景:

优化建议:通病

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( 'al数码组' ) and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 88 , 10 ) and year(dg.s_added_time)!='1970' and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## 62b7bf23a096d049ffa0424cb43b3a93

耗时:1-11秒

使用场景:库存异常导出

优化建议:sql没空间了,得业务改造

```sql
select dso.skc as skc, dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid, dsos.order_amount as orderamount, dsos.size as size, dso.add_time as addtime, dso.order_status_value as orderstatus, dge.stock_b as stockb, dge.wait_ontheshelf as waitontheshelf, dge.sales_volume as salesvolume from dms_sub_order dso left join dms_goods dg on dg.skc = dso.skc left join dms_sub_order_size dsos on dsos.sub_order_id = dso.sub_order_id left join dms_goods_extend dge on dge.skc = dso.skc and dge.size = dsos.size where dso.is_delete = 0 and dg.is_delete = 0 and dsos.is_delete = 0 and dge.is_delete = 0 and dsos.order_amount > 0 and dso.system_flag = 2 and dso.order_status_value in ( 23 , 15 , 14 , 13 , 12 , 11 , 10 , 9 , 8 , 7 ) and dso.order_mark_value not in ( '812' , '804' , '816' , '831' , '736' ) and dge.warehouse_inventory_sale_rate_b > 0 and dge.warehouse_inventory_sale_rate_b <= 1 and ((dso.prepare_type_value = 0 and dso.order_level = 0) or ( dso.prepare_type_value = 9 and dso.order_level = 1)) limit 125000,5000;
```

![image-20210114103917405](/Users/lumac/Library/Application Support/typora-user-images/image-20210114103917405.png)

## d3626b02114d3e246bf066f2b4ecf535

耗时:

使用场景:

优化建议:通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al数码组' ) and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 88 , 10 ) and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## 4dec3fe730f2ab6aba6bb8ecf0a176d6

耗时:

使用场景:

优化建议:通病

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( '市场男装沙河组' , '沙河二组' , '沙河三组' ) and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-22' and dg.goods_level_id in ( 61 , 10 ) and year(dg.s_added_time)!='1970' and dg.maintenance_uid in ( '市场中东成衣组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '流花男装组' , '样衣' );
```



## 22e97f5d07d8453beea3ef0cb8cbeedc

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( '沙河内衣组' , '沙河一组' ) and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-22' and dg.goods_level_id in ( 61 , 10 ) and year(dg.s_added_time)!='1970';
```



## 29af85ef95f9f65f43a8b205b1fab9561

耗时:

使用场景:

优化建议:业务限制下,这种count太大

```sql
select count(1) from dms_goods where is_delete = 0 and show_status = 2;
```



## 8b16a56faa14797ea9342a87ce10d415

耗时:

使用场景:

优化建议:

```sql
select count(1) from dms_goods where is_delete = 0 and show_status = 2;
```



## 8cc59f84dcc5979d3279927f9236c366

耗时:

使用场景:

优化建议:

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( '沙河内衣组' , '沙河一组' ) and dg.sales_volume <= '99' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-22' and dg.goods_level_id in ( 61 , 10 );
```



## 67a00600a38090c52655a5a6443ec791

耗时:

使用场景:

优化建议:

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( '市场男装沙河组' , '沙河二组' , '沙河三组' ) and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-22' and dg.goods_level_id in ( 61 , 10 ) and dg.maintenance_uid in ( '市场中东成衣组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '流花男装组' , '样衣' );
```



## 77c1d95baf5e2aa7d9c82ed8cbaeb338

耗时:

使用场景:

优化建议:

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( 'al数码组' ) and dg.sales_volume <= '99' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 88 , 10 ) and year(dg.s_added_time)!='1970' and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## bce32a7cffdb0ccdb7c584a96b80909d

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.warn_inventory_status = '1' and dg.goods_level_id in ( 61 ) and dg.deposit_sale_ratio_total <= '1' and dg.maintenance_uid in ( 'romwe单独款' );
```



## 60ef8e62df8d54c9517f4c41a4fe3d85

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '200' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-06-12' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1740 , 1912 , 2709 );
```



## 7405220928591c932824f11a2a1f0368

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '60' and dg.sales_volume <= '100' and year(dg.s_added_time)!='1970' and s_added_time >= '2020-12-22' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1727 , 1733 , 1738 , 1779 , 2214 , 1732 , 1740 , 1871 , 1912 , 2709 , 1773 , 1860 , 1882 , 1922 , 1929 , 1930 , 1931 , 1932 , 1933 , 1934 , 1935 , 2050 , 1739 , 2676 , 1735 , 1776 , 1780 , 2167 , 1734 , 2208 , 2209 , 2210 , 2211 , 2212 , 2213 , 1862 , 2195 , 2196 , 2197 , 2198 , 2199 , 2261 , 2490 , 2205 , 2311 , 2312 , 2314 , 2315 , 2316 , 2317 , 1880 , 2200 , 2201 , 2202 , 2203 , 2204 , 2310 , 2313 , 2318 , 2319 , 2320 , 2321 , 1866 , 2191 , 2192 , 2278 , 2279 , 1878 , 2174 , 2175 );
```



## 3a64fac4abee904adf3d73e22acb5316

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al数码组' ) and dg.sales_volume <= '99' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 88 , 10 ) and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## f35c465f485404eb0ed0af2af786b558

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '200' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-09-24' and s_added_time >= '2020-06-12' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1732 , 1740 , 1871 , 1912 , 2709 );
```



## 2078c8ee8f7628a7d8a10c9b8a57db6e

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.skc in ( 'swvest02190605527' ) and dg.maintenance_uid in ( '市场家居南通组' , '常熟一组' , '市场大码十三行组' , '十三行一组' , '市场男装十三行组' );
```



## dfd4b0f57d447d3f1d4563171da6eed3

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '350' and dg.sales_volume <= '450' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-09-20' and s_added_time >= '2020-06-11' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1732 );
```



## f41b739c916fa940684c5052d5afc41b

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al饰品组' , '市场家居阿里组' , 'al儿童非成衣组' , 'al男士非成衣组' ) and dg.sales_volume >= '100' and dg.warn_inventory_status = '1' and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 10 , 61 ) and dg.maintenance_uid in ( '市场家居阿里组' , 'al女士鞋子组' , 'al儿童非成衣组' , 'al男士非成衣组' , 'al彩妆组' , 'al订单组' , 'al数码组' , '市场成衣订单组' , 'al饰品组' , '市场女鞋一组' );
```



## ca5668729a85401910c6278139113ca5

耗时:2856ms

使用场景:

优化建议:全表扫描,要结合业务来优化了

```sql
select count(dg.id) from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id where dg.is_delete = 0 and concat(ds.type, '-', ds.category_id) in ('1-146','1-145');
```

![image-20201229145301126](/Users/lumac/Library/Application Support/typora-user-images/image-20201229145301126.png)

## e62d1b191d279626d554a3c9dc9c3bef

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '25' and year(dg.s_added_time)!='1970' and s_added_time >= '2020-12-23' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1727 , 1733 , 1738 , 1779 , 2214 , 1732 , 1740 , 1871 , 1912 , 2709 , 1773 , 1860 , 1882 , 1929 , 1930 , 1931 , 1932 , 1933 , 1934 , 1935 , 2050 , 1739 , 2676 , 1735 , 1776 , 1780 , 1734 , 2208 , 2209 , 2210 , 2211 , 2212 , 2213 , 1862 , 2195 , 2197 , 2199 , 2261 , 2490 , 2205 , 2311 , 2312 , 2314 , 2315 , 2316 , 2317 , 1880 , 2200 , 2201 , 2202 , 2203 , 2204 , 2310 , 2313 , 2318 , 2319 , 2320 , 2321 );
```



## 16b73e7807bdc7bd3c32e5e8e60a0e90

耗时:7110ms

使用场景:

优化建议:

count:79339

```sql
select count(*) from dms_goods_management dgm left join dms_goods dg on dgm.skc = dg.skc where dgm.is_delete = 0 and dg.maintenance_uid in ( 'al订单组' , 'al泳衣组' , 'al内衣组' ) and dgm.change_source=3;
```

![image-20201229145446100](/Users/lumac/Library/Application Support/typora-user-images/image-20201229145446100.png)

## 0a3222d7d139f44cd3cab9cc2b2a8c6f

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( '沙河一组' , '沙河内衣组' ) and year(dg.s_added_time)!='1970' and s_added_time <= '2020-12-21' and dg.goods_level_id in ( 61 , 10 );
```



## 4c83ae3675b2bd2176506b71839b238f

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '500' and dg.goods_level_id in ( 107 , 109 , 105 , 87 , 61 , 55 , 10 ) and dg.category_id in ( 1733 );
```



## 81b18f4134494d9eae804f7ca918bcd3

耗时:7539ms

使用场景:

优化建议:sql太诡异

count:273196

```sql
select count(t1.id) from dms_activity_skc_relation t1 join dms_activity_promotion t2 on t1.promotion_id = t2.promotion_id where t2.is_del = 0 and (t2.begin_time >= '2020-11-29 00:00:00' or t2.begin_time = '1970-01-01 08:00:01') and t1.is_del = 0;
```

![image-20201229145716325](/Users/lumac/Library/Application Support/typora-user-images/image-20201229145716325.png)

## 2093e513eb37b0923c5a3374ef7ba0f9 准备上线

耗时:1925ms

使用场景:

优化建议:重写sql

```sql
select max(id) from dms_goods_extend where is_delete = 0;
```

```sql
select id from dms_goods_extend order by id desc limit 1;
```

类似的校验和:e0b50721457b4ea1a262f7f3d1f764fd,7432f1c8f928b240791c58e8f689925a

## 76f508f79e5a1378c5c5b908059c259a 已解决

耗时:3.59

使用场景:

优化建议:skc太多,分批处理吧

```sql
select skc, s_added_time, r_added_time from dms_goods where skc in ()
```



## 3204fcb5170365c2985839180816df99

耗时:3038ms

使用场景:

优化建议:sql无空间,业务也不好修改

count:1033240

```sql
select count(dg.id) from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id where dg.is_delete = 0 and concat(ds.type, '-', ds.category_id) in ('1-144','1-143','1-142','1-149','1-91','1-90','1-13','2-69','1-11','2-68','1-99','2-67','1-12','1-97','1-10','1-98','2-63','1-96','1-152','1-151','1-150','1-159','2-70','1-20','1-124','1-123','1-19','1-122','1-17','1-18','1-240','2-1','1-15','2-2','1-16','1-249','1-248','1-247','1-30','1-253','1-28','1-131','1-29','1-250','1-27','1-46','1-47','1-181','1-180','1-40','1-41','1-223','1-102','1-189','1-222','1-221','1-188','1-187','1-39','1-186','1-1','1-185','1-2','1-184','1-38','1-183','1-3','1-4','1-5','1-109','1-229','1-6','1-108','1-7','1-8','1-227','1-9','1-226','1-104','1-225','1-103','1-224','1-193','1-58','1-192','1-191','1-190','1-54','1-51','1-113','1-112','1-48','1-195','1-49','1-194','1-119','1-118','1-239','1-117','1-116','1-115','1-236','1-235','1-160','1-68','1-69','1-66','1-67','1-65','1-201','1-164','1-163','1-161','1-209','1-208','1-207','1-206','1-205','1-203','1-169','1-72','1-79','1-171','1-170','1-78','1-75','1-74','1-179','1-212','1-211','1-178','1-210','1-177','1-176') and dg.goods_level_id in (33,65,67,4,69,101,102,39,103,7,8,105,106,76,79,83,54,93);
```



## cb03cd77605f008633e322edb0756815

耗时:3770ms

使用场景:

优化建议:add_time添加索引

```sql
select count(o.skc) from dms_rec_order o where o.is_delete = 0 and o.add_time >= '2020-12-28 00:00:00' and o.add_time <= '2020-12-28 23:59:59';
```

![image-20201229150225086](/Users/lumac/Library/Application Support/typora-user-images/image-20201229150225086.png)

## 7b12d92cbfad2883079ec8c77fa0873d

耗时:

使用场景:清0操作

优化建议:加个前置条件

```sql
update dms_unsalable_goods set is_delete = 1 where bi_effect_date < '2020-12-29 00:00:00' and is_delete = 0;
```



## 022a3a5d3f9cd5a65c7d18828bd19ddc 已解决

耗时:3-4秒

使用场景:

优化建议:supplierId太多了

```sql
select id, show_status, skc, s_sale_status, r_sale_status, s_sale_by_inventory, r_sale_by_inventory, s_added_time, r_added_time, s_gross_profit_margin, r_gross_profit_margin, goods_thumb, average_return_time, goods_level_id, maintenance_uid, short_in_skc_status, short_in_size_status, warn_inventory_status, delivery_time, supplier_id, supplier_code, production_team_id, production_team_name, season_id, deposit_sale_ratio, sales_volume, outofstock_total, first_order_date, stock_date, bi_effect_date, add_time, update_time, add_uid, update_uid, is_delete, last_update_time, first_order_id, order_count, deposit_sale_ratio_total, category_id, s_gross_profit_margin_week, r_gross_profit_margin_week, product_time, jit_receipt_time, is_sale_attribute, spu from dms_goods where show_status = 2 and is_delete = 0 and supplier_id in()
```



## 11879d49362562070cae8674e3e2f785

耗时:1571ms

使用场景:

优化建议:sql无优化空间

count:4438

```sql
select count(1) from dms_order where is_delete = 0 and order_status_value in ( 2 , 3 ) and order_type = 1 and prepare_type_value not in ( '12' );
```

![image-20201229150626745](/Users/lumac/Library/Application Support/typora-user-images/image-20201229150626745.png)

## ae5975229fc40d8a0a9cdee853a43be9 

耗时:4-8秒

使用场景:jit数据统计

优化建议:count太多,可以把order by去掉

```sql
select count(*) from (select s.id as id, s.supplier_id as supplierid, s.skc as skc ,s.goods_type as goodstype, e.id as eid,e.jit_available_num as jitavailablenum, e.require_delivery_num as requiredeliverynum, e.size as size from dms_jit_stock s left join dms_jit_stock_extend e on e.jit_id = s.id left join dms_goods g on g.skc = s.skc left join dms_supplier si on si.id = g.supplier_id where s.is_delete = 0 and e.is_delete = 0 and e.jit_available_num > 0 and g.sales_volume >= 0 and si.id = g.supplier_id and s.goods_type = 2 order by skc)a;
```

![image-20210118113218897](/Users/lumac/Library/Application Support/typora-user-images/image-20210118113218897.png)

## 1fccf6ace0ec49e6c558df5d0ccf1494 1.19号上线 已解决

耗时:3898ms

使用场景:countNoSalesVolumeGoodsByQuery

优化建议:索引优化

count:68853

```sql
select count(id) from dms_goods where sales_volume = 0 and goods_level_id in ( 66 , 68 , 70 , 10 , 75 , 12 , 77 , 78 , 80 , 81 , 84 , 86 , 87 , 88 , 25 , 89 , 90 , 91 , 92 , 94 , 96 , 33 , 97 , 104 , 105 , 42 , 43 , 47 , 51 , 53 , 55 , 58 , 61 , 62 ) and id>0;
```

![image-20201229153827906](/Users/lumac/Library/Application Support/typora-user-images/image-20201229153827906.png)

优化后的sql

```sql
SELECT COUNT(id)
FROM dms_goods force index(idx_goodsLevelId)
WHERE goods_level_id IN (66, 68, 70, 10, 75, 12, 77, 78, 80, 81, 84, 86, 87, 88, 25, 89, 90, 91, 92, 94, 96, 33, 97, 104, 105, 42, 43, 47, 51, 53, 55, 58, 61, 62)
	AND id > 0 
    and  sales_volume = 0;
```



## 54198cc240ef43a788af317eee53e2e4

耗时: 5.02

使用场景:尾货表删除

优化建议:分批删除吧

n个一批

```sql
delete from dms_tail_extend where 1=1;
```



## b7989ef0908b4ff8b6a9215993a447ba

耗时:1-7秒

使用场景:

优化建议:加skc排序

```sql
select dglaa.* from dms_goods_level_auto_alteration dglaa left join dms_goods dg on dglaa.skc = dg.skc where dglaa.is_delete = 0 and dglaa.is_auto_generate = 1 and dglaa.status = 1 and dglaa.maintenance_uid in ( 6 ) and dg.goods_level_id in ( 4 , 10 , 12 , 25 , 43 , 44 , 45 , 46 , 47 , 51 , 52 , 55 , 61 , 62 , 66 , 67 , 68 , 70 , 75 , 77 , 79 , 80 , 81 , 83 , 84 , 85 , 86 , 87 , 88 , 89 , 90 , 91 , 92 , 93 , 94 , 95 , 96 , 97 , 99 , 100 , 101 , 102 , 103 , 104 , 105 , 106 , 107 , 108 , 109 ) order by dglaa.sales_volume desc limit 0,100;
```

![image-20201229154843397](/Users/lumac/Library/Application Support/typora-user-images/image-20201229154843397.png)

## 688b7cadf006b4d8d3f2ec5fdecb6ed8

耗时:

使用场景:

优化建议:只要178ms

```sql
SELECT dglaa.*
FROM dms_goods_level_auto_alteration dglaa
	LEFT JOIN dms_goods dg ON dglaa.skc = dg.skc
WHERE dglaa.is_delete = 0
	AND dglaa.is_auto_generate = 1
	AND dglaa.status = 1
	AND dglaa.maintenance_uid IN (6)
	AND dg.goods_level_id IN (4, 10, 12, 25, 43, 44, 45, 46, 47, 51, 52, 55, 61, 62, 66, 67, 68, 70, 75, 77, 79, 80, 81, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109)
ORDER BY dglaa.sales_volume DESC,dglaa.skc desc
LIMIT 0, 100;
```



## ef8fe29b9a60554e9f00a822f34bcabf

耗时:5818ms

使用场景:

优化建议:全表扫描了

```sql
select sum( case when g.short_in_skc_status=2 then 1 else 0 end ) shortinskcnum, sum( case when g.short_in_size_status=2 then 1 else 0 end ) shortinsizenum,g.maintenance_uid from dms_goods g left join dms_goods_extra ex on ex.skc=g.skc where g.is_delete = 0 and g.show_status = 2 and g.maintenance_uid in ( 'al儿童非成衣组' , 'al内衣组' , 'al女士鞋子组' , 'al彩妆组' , 'al成衣一组' , 'al成衣二组' , 'al数码组' , 'al泳衣组' , 'al男士非成衣组' , 'al睡衣组' , 'al订单组' , 'al饰品组' , 'romwe单独款' , 'vmi' , '十三行一组' , '品牌vmi组' , '外协一组' , '外协三组' , '外协二组' , '外协内衣组' , '外协大码组' , '外协牛仔组' , '外协男装组' , '市场大码十三行组' , '市场大码沙河组' , '市场大码阿里组' , '市场女鞋一组' , '市场孕妇装' , '市场家居南通组' , '市场家居阿里组' , '市场成衣订单组' , '市场男装沙河组' , '市场男装阿里组' , '市场童装阿里组' , '常熟一组' , '沙河一组' , '沙河三组' , '沙河二组' , '沙河内衣组' , '生产中东组' , '生产大码组' , '生产孕妇装' , '生产男装组' , '生产童装一组' , '生产童装三组' , '生产童装二组' , '生产维护一组' , '生产维护七组' , '生产维护三组' , '生产维护九组' , '生产维护二组' , '生产维护五组' , '生产维护八组' , '生产维护六组' , '生产维护十组' , '生产维护四组' , '美国市场款鞋子' , '美国鞋子' ) group by g.maintenance_uid;
```

![image-20201229155045073](/Users/lumac/Library/Application Support/typora-user-images/image-20201229155045073.png)

## 0926df43e907b2efc3ac81ef907b3448

耗时:3671ms

使用场景:

优化建议:备货优化的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.goods_level_id in ( 61 ) and dg.maintenance_uid in ( '市场男装阿里组' , '市场大码阿里组' , '市场童装阿里组' , 'al睡衣组' , '市场礼服组' );
```



## 0a51a9e04139be2c2a4676eea3ca90c4

耗时:2224ms

使用场景:滞销管理页面count

优化建议: 可以不用统计主表的数据,直接统计扩展表就行了

count:56114

```sql
select count(*) from dms_unsalable_goods dug left join dms_unsalable_goods_extend duge on duge.unsalable_goods_id = dug.id where dug.is_delete = 0 and duge.is_delete = 0;
```

![image-20201229155407867](/Users/lumac/Library/Application Support/typora-user-images/image-20201229155407867.png)

```sql
    SELECT COUNT(*)
FROM dms_unsalable_goods_extend dug where  dug.is_delete = 0;
```

查询时间：51ms

## 0caf7790cee64b99c8d26c96617d4369

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( '生产男装组' , '生产童装组' , '生产维护三组' , '生产维护二组' , '生产维护一组' , '生产童装一组' , '生产童装二组' , '生产维护十组' , '生产童装三组' );
```



## d377e618755b54fb34c45f921aa6bebb

耗时:2147ms

使用场景:

优化建议:重写sql吧

count:126811

```sql
select count(size_id) from ( select distinct(doos.id) size_id from dms_order doo join dms_order_size doos on doo.id=doos.order_id and doos.order_amount>0 where doo.id in( select distinct dr.order_id from dms_sub_order do left join dms_order_sub_relation dr on do.sub_order_id = dr.sub_order_id and dr.is_del = 0 where do.is_delete = 0 and do.first_mark_value = '1' and do.add_time >= '2020-12-21 00:00:00' and do.add_time <= '2020-12-27 23:59:59' and do.show_flag = 0 and do.order_status_value in ( 7 , 8 , 9 , 10 , 11 , 12 , 13 , 14 , 15 , 20 , 21 , 22 , 23 , 24 , 16 , 17 ) ) ) uniontable;
```

![image-20201229155831886](/Users/lumac/Library/Application Support/typora-user-images/image-20201229155831886.png)

## 6494f4e04acc09ab09af4287eecd1768

耗时:dbadmin不耗时了,再观望

使用场景:

优化建议:

```sql
SELECT id
FROM dms_goods
WHERE show_status = 2
	AND maintenance_uid = 'vmi'
ORDER BY id;
```

![image-20201229160105851](/Users/lumac/Library/Application Support/typora-user-images/image-20201229160105851.png)

## 103b8d7af323428ba0d021b9fef339f4

耗时:

使用场景:

优化建议:备货标签通病

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( 'al女士鞋子组' , '市场女鞋一组' ) and dg.sales_volume <= '99' and dg.goods_level_id in ( 61 , 10 );
```



## 2f273e839334cabb93967cf553c20601

耗时:

使用场景:

优化建议:备货标签通病

```sql
select count(distinct dg.skc,b.size) from dms_goods dg left join dms_goods_extend b on b.skc = dg.skc left join dms_supplier s on dg.supplier_id=s.id left join dms_goods_expand e on e.skc=dg.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.is_delete = 0 and dg.show_status = 2 and b.status = 1 and b.is_delete = 0 and dg.maintenance_uid in ( 'al女士鞋子组' , '市场女鞋一组' ) and dg.sales_volume >= '100' and dg.goods_level_id in ( 61 , 10 );
```



## 订单审核 已优化

耗时:1341ms

使用场景:

优化建议:索引优化

count:165

```sql
select count(1) from dms_order do left join dms_goods dg on do.skc = dg.skc where `do`.is_delete=0 and do.maintenance_name in ( '市场中东成衣组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '流花男装组' , '样衣' ) and do.add_uid = '周丽2' and `do`.prepare_type_value in ('0','11','2','3','4','7','8','9') and dg.goods_level_id in ( 10 , 61 ) and `do`.order_status_value in ( '2' , '3' );
```

![image-20201229160347084](/Users/lumac/Library/Application Support/typora-user-images/image-20201229160347084.png)

## 7432f1c8f928b240791c58e8f689925a 已优化

耗时:2199ms

使用场景:实时销量获取最大id

优化建议:

count:2159615

```sql
select max(id) from dms_goods where is_delete = 0;
```

![image-20201229160441145](/Users/lumac/Library/Application Support/typora-user-images/image-20201229160441145.png)

优化后的sql:耗时567ms

```sql
select id from dms_goods  where is_delete = 0 order by id desc limit 1;
```



## c903259789dcead2688b42ad105ee10f

耗时:

使用场景:

优化建议:备货建议表的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al女士鞋子组' , '市场女鞋一组' ) and dg.sales_volume >= '100' and dg.goods_level_id in ( 61 , 10 );
```



## 6b07c0f3efe2a979a81e486115915a36

耗时:2秒 实际dbadmin5ms

使用场景: vim数据查询

优化建议:

```sql
select id, show_status, skc, s_sale_status, r_sale_status, s_sale_by_inventory, r_sale_by_inventory, s_added_time, r_added_time, s_gross_profit_margin, r_gross_profit_margin, goods_thumb, average_return_time, goods_level_id, maintenance_uid, short_in_skc_status, short_in_size_status, warn_inventory_status, delivery_time, supplier_id, supplier_code, production_team_id, production_team_name, season_id, deposit_sale_ratio, sales_volume, outofstock_total, first_order_date, stock_date, bi_effect_date, add_time, update_time, add_uid, update_uid, is_delete, last_update_time, first_order_id, order_count, deposit_sale_ratio_total, category_id, s_gross_profit_margin_week, r_gross_profit_margin_week, product_time, jit_receipt_time, is_sale_attribute, spu from dms_goods where show_status = 2 and maintenance_uid = 'vmi' and is_delete = 0;
```



## 9ffe5cd081f4270afa06a63c849e80dd

耗时:6248ms

使用场景:报损单页面count

优化建议:sql没有优化的空间了,看看业务能否加以限制了,有一个优化的场景,可以把notin去掉,这边是多余的操作的

```sql
select count(1) from dms_goods_scrap_order o left join dms_goods g on g.skc=o.skc where o.is_del=0 and g.is_delete=0 and o.order_status not in (0, 9) and o.order_status in ( 1 , 2 , 3 , 4 , 6 ) and o.add_time >= '2020-09-28 00:00:00' and o.add_time <= '2020-12-28 23:59:59';
```

![image-20201229161204219](/Users/lumac/Library/Application Support/typora-user-images/image-20201229161204219.png)

## 96ceaad6f9389725c55dc980a0f24c81 已搞定

耗时: 8.58

使用场景:全部订单导出,查询下单时间

优化建议:分批查询

```sql
select date_format( do.add_time, '%y-%m-%d' ) as add_time from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 );
```

![image-20201229161549030](/Users/lumac/Library/Application Support/typora-user-images/image-20201229161549030.png)

## 2a8973c42b9dc944e05044433a9942d2

耗时:

使用场景:

优化建议:同上

```sql
select date_format( do.add_time, '%y-%m-%d' ) as add_time from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.show_flag = 0 and do.order_status_value in ( 8 , 20 , 21 , 22 , 23 );
```



## 442f1ce1c18f83f7f04f4016c782f11f

耗时:

使用场景:

优化建议:备货标签的通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al成衣二组' ) and dg.sales_volume >= '100' and dg.sales_volume <= '300' and dg.goods_level_id in ( 61 ) and dg.deposit_sale_ratio_total <= '4';
```



## d819f8fe86c6e16f41a511e1e1ac545d

耗时:

使用场景:

优化建议:同上

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al成衣二组' ) and dg.sales_volume >= '300' and dg.goods_level_id in ( 61 ) and dg.deposit_sale_ratio_total <= '4';
```



## 984963fb8d437877542e433274265585

耗时:

使用场景:

优化建议:同上

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al成衣二组' ) and dg.sales_volume >= '300' and dg.goods_level_id in ( 61 ) and dg.deposit_sale_ratio_total <= '4';
```



## a109de339a5d0f22bea8ac8c0ea79ecb

耗时:3156ms

使用场景:

优化建议:数据量太大,索引无用武之地

```sql
select count(1) from ( select dos.id from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.prepare_type_value = '9' and do.add_time >= '2020-09-30 00:00:00' and do.add_time <= '2020-12-29 23:59:59' and is_production_completion = '1' and do.show_flag = 0 and do.order_status_value in ( 8 , 15 , 20 , 21 , 22 ) ) t;
```

![image-20210107150154510](/Users/lumac/Library/Application Support/typora-user-images/image-20210107150154510.png)

## c4aeefd90eceee8095951f4d18c272f1

耗时:2870ms

使用场景:jit任务查询订单详情

优化建议:业务改造,改成游标取值,这边次数比较少,这个需求先不动

```sql
select s.id as id, s.supplier_id as supplierid, s.skc as skc, s.goods_type as goodstype, e.id as eid, e.jit_available_num as jitavailablenum, e.require_delivery_num as requiredeliverynum, e.size as size, s.skc as eskc from dms_jit_stock s left join dms_jit_stock_extend e on e.jit_id = s.id left join dms_goods g on g.skc = s.skc left join dms_supplier si on si.id = g.supplier_id where s.is_delete = 0 and e.is_delete = 0 and e.jit_available_num > 0 and g.sales_volume >= 0 and si.type = 2 order by skc limit 90000,10000;
```

![image-20201229161941436](/Users/lumac/Library/Application Support/typora-user-images/image-20201229161941436.png)

## 30c24d84ac606602bb2da89e99b1ab93

耗时:

使用场景:

优化建议:通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al女士鞋子组' , '市场女鞋一组' ) and dg.warn_inventory_status = '1' and dg.goods_level_id in ( 61 );
```



## 4c9335af407eac5014b6f184446900a8

耗时:

使用场景:

优化建议:通病

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.maintenance_uid in ( 'al女士鞋子组' ) and dg.sales_volume >= '500' and dg.warn_inventory_status = '1' and dg.goods_level_id in ( 61 , 10 );
```

## d41e6158ab97a2d4221572bd9992c87e  准备上线中

使用场景:新商品层次查询标签数据

优化建议:分批查找 能优化91次

```sql
SELECT `ds` . `id` , `ds` . `skc` , `dg` . `title` `labeltitle` , `dgsl` . `title` `labeltypetitle` , `ds` . `label_type_id` , `ds` . `label_id` FROM `dms_skc_rel_label` `ds` LEFT JOIN `dms_goods_stock_label` `dg` ON `dg` . `id` = `ds` . `label_id` LEFT JOIN `dms_goods_stock_label` `dgsl` ON `dgsl` . `id` = `dg` . `pid` WHERE `ds` . `is_del` = ? AND `dg` . `is_del` = ? AND `dg` . `status` = ? AND `dgsl` . `status` = ? AND `ds` . `skc` IN (...) ORDER BY `ds` . `label_type_id` , `ds` . `label_id` , `ds` . `id` DESC ;
```

使用场景:新商品层次查询标签数据

优化建议:分批查找 能优化91次



## 04bbdff7a1e4167be45312ef4a3dade7 已解决

耗时:4.75秒

使用场景:新商品层次查询国家销量数据

优化建议:分批查找 能优化89次

```sql
select id,skc,size,sales_volume,country from dms_country_sale where is_delete=0 and skc in()
```



## 9c7be1f97ea13a1fcaede0e89057a179

耗时:

使用场景:备货标签的通病

优化建议:

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '100' and year(dg.s_added_time)!='1970' and s_added_time >= '2020-11-23' and dg.goods_level_id in ( 10 , 61 ) and dg.supplier_id in ()
```



## 2945303dd317f8ca2960567fc69e2e7d

耗时:4105ms

使用场景:异常订单表count数量

优化建议:业务上限制时间吧,时间跨度太大

```sql
select count(distinct dso.id) from dms_sub_order dso join dms_exception_order deo on deo.sub_order_id = dso.sub_order_id where deo.type = 2 and deo.exception_type = 12 and deo.maintenance_uid in ( '36' ) and dso.add_time >= '2020-09-30 00:00:00' and dso.add_time <= '2020-12-29 23:59:59' and deo.status ='1' and dso.order_status_value in ( '8' ) and dso.is_delete = 0 and deo.is_delete = 0 and dso.sub_order_id != '';
```



## 8389ba732a9c6f81d7241e9d87b7f197

耗时:13082ms

使用场景:subOrderMapper.queryProductJitOrder() jit查询超期的单子

优化建议:业务改造,分批查询

```sql
SELECT *
FROM dms_sub_order
WHERE prepare_type_value = 9
	AND delivery_tag = 1
	AND system_flag = 2
	AND order_mark_value = 733
	AND order_status_value IN (7, 8, 9, 10, 11, 12, 13, 14, 15)
	AND is_delete = 0
	AND order_level = 1
	AND '2021-01-07 13:17:17' > demand_receipt_date
ORDER BY skc;
```



## 4b24bd36fc48c1d355adbca08f7c2684

耗时:5秒

使用场景:备货标签页面默认全部count

优化建议:全表扫描了,业务改造,加一些筛选条件

```sql
select count(1) from dms_skc_rel_label ds where ds.is_del=0;
```

![image-20210107144411686](/Users/lumac/Library/Application Support/typora-user-images/image-20210107144411686.png)

## 42b3f8417bddf839ce975223d9a5edce 已优化

耗时:20314ms

使用场景:下单审核count页面

优化建议:强制走order_status_value的索引.dg这张表也是不应该关联的,如果没有条件就不关联

count:38

```sql
select count(1) from dms_order do left join dms_goods dg on do.skc = dg.skc where `do`.is_delete=0 and do.add_uid = '叶晓飞' and `do`.prepare_type_value in ('0','11','2','3','4','7','8','9') and `do`.order_type = '1' and `do`.order_status_value in ( '2' , '3' );
```

![image-20201229143422164](/Users/lumac/Library/Application Support/typora-user-images/image-20201229143422164.png)

## 5ff84f2baf14dfe2894df9ee8875aaec

耗时:10秒

使用场景:子母单导出,选出下单时间

优化建议:分批查询

```sql
select date_format( do.add_time, '%y-%m-%d' ) as add_time from dms_sub_order do left join dms_sub_order_size dos on do.sub_order_id=dos.sub_order_id and dos.order_amount>0 where do.is_delete = 0 and do.prepare_type_value = '9' and do.add_time >= '2020-09-30 00:00:00' and do.add_time <= '2020-12-29 23:59:59' and is_production_completion = '1' and do.show_flag = 0 and do.order_status_value in ( 8 , 15 , 20 , 21 , 22 );
```



## 0bed4b8d4825a720e73542dab3268bc1

耗时:2652ms

使用场景:频率不高

优化建议:业务设计的不合理,没有的数据应该是用0,不应该就没有,这样sql极其难写

```sql
select dgcp.skc from dms_goods_clearance_pool dgcp left join dms_goods dg on dg.skc = dgcp.skc where dgcp.is_del = 0 and dg.show_status != 2 and dgcp.skc not in (select distinct skc from dms_unsalable_goods where is_delete = 0);
```

![image-20201229143254694](/Users/lumac/Library/Application Support/typora-user-images/image-20201229143254694.png)

left join优化

```sql
select dgcp.skc
from dms_goods_clearance_pool dgcp
         left join dms_goods dg on dg.skc = dgcp.skc
         left join (select * from dms_unsalable_goods where is_delete = 0) dug on dug.skc = dg.skc
where dgcp.is_del = 0
  and dg.show_status != 2
  and dug.id is null;
```



## 0e6d02e1042a19099369319879281c8e

耗时:分析过了,雷同sql

使用场景:

优化建议:

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.sales_volume >= '100' and dg.sales_volume <= '499' and year(dg.s_added_time)!='1970' and s_added_time >= '2020-11-23' and dg.goods_level_id in ( 10 , 61 ) and dg.supplier_id in()
```

## VMI历史备货建议count180  搞定

```sql
select count(1) from dms_vmi_stock_history h left join dms_supplier s on s.id=h.supplier_id where h.is_delete=0 and h.generate_time >='2020-12-15 00:00:00' and h.generate_time <='2020-12-30 23:59:59';
```

业务查询限制一天

## 66f3a56e913f7d5210d6399e563c316a

耗时:1-4秒

使用场景:List<GoodsEntity> scanGoods(InventoryWarningCursor inventoryWarningCursor);

```sql
SELECT DISTINCT goods.*
FROM dms_goods goods
WHERE goods.id > 0
	AND goods.is_delete = 0
	AND goods.show_status = 2
	AND goods.warn_inventory_status = 1
	AND goods.maintenance_uid IN (
		'al男士非成衣组', 
		'美国', 
		'常熟一组', 
		'市场女鞋一组', 
		'al彩妆组', 
		'市场泳衣二组', 
		'市场男装十三行组', 
		'al数码组', 
		'市场礼服组', 
		'市场成衣订单组', 
		'美国鞋子', 
		'美国市场款鞋子', 
		'al女士鞋子组', 
		'al睡衣组', 
		'市场童装阿里组', 
		'市场男装阿里组', 
		'市场大码阿里组', 
		'al成衣二组', 
		'市场家居阿里组', 
		'al成衣一组', 
		'市场男装沙河组', 
		'十三行一组', 
		'市场大码沙河组', 
		'欧洲', 
		'市场大码十三行组', 
		'沙河一组', 
		'沙河三组', 
		'沙河二组', 
		'市场大码订单组', 
		'al内衣组', 
		'沙河内衣组', 
		'al泳衣组', 
		'市场孕妇装', 
		'al订单组', 
		'al饰品组', 
		'市场家居南通组', 
		'样衣', 
		'al儿童非成衣组'
	)
	AND goods.goods_level_id IN (10, 47, 70)
	AND goods.category_id IN (1875, 1778, 1772, 1872, 1901, 2095, 1916, 1917, 1914, 2371, 1899, 1920, 1768, 1771, 1775, 1770, 2589, 2590)
ORDER BY goods.id ASC
LIMIT 100;
```

优化后:利用filesort

```sql
SELECT DISTINCT goods.*
FROM dms_goods goods
WHERE goods.id > 0
	AND goods.is_delete = 0
	AND goods.show_status = 2
	AND goods.warn_inventory_status = 1
	AND goods.maintenance_uid IN (
		'al男士非成衣组', 
		'美国', 
		'常熟一组', 
		'市场女鞋一组', 
		'al彩妆组', 
		'市场泳衣二组', 
		'市场男装十三行组', 
		'al数码组', 
		'市场礼服组', 
		'市场成衣订单组', 
		'美国鞋子', 
		'美国市场款鞋子', 
		'al女士鞋子组', 
		'al睡衣组', 
		'市场童装阿里组', 
		'市场男装阿里组', 
		'市场大码阿里组', 
		'al成衣二组', 
		'市场家居阿里组', 
		'al成衣一组', 
		'市场男装沙河组', 
		'十三行一组', 
		'市场大码沙河组', 
		'欧洲', 
		'市场大码十三行组', 
		'沙河一组', 
		'沙河三组', 
		'沙河二组', 
		'市场大码订单组', 
		'al内衣组', 
		'沙河内衣组', 
		'al泳衣组', 
		'市场孕妇装', 
		'al订单组', 
		'al饰品组', 
		'市场家居南通组', 
		'样衣', 
		'al儿童非成衣组'
	)
	AND goods.goods_level_id IN (10, 47, 70)
	AND goods.category_id IN (1875, 1778, 1772, 1872, 1901, 2095, 1916, 1917, 1914, 2371, 1899, 1920, 1768, 1771, 1775, 1770, 2589, 2590)
ORDER BY goods.id ASC,goods.skc desc
LIMIT 100;
```

## 036fbb30a1cb537b8b0269ea72cfd931

使用场景:查询有在途的skc数量

count返回值:5499

```sql
SELECT COUNT(t3.skc)
FROM (
	SELECT t1.skc, SUM(t1.ontheway) AS totalontheway
	FROM `dms_goods_on_the_way` t1
		JOIN dms_goods t2 ON t1.skc = t2.skc
	WHERE t1.ontheway_type IN (1, 2, 3, 4)
		AND t2.show_status = 1
		AND t2.goods_level_id IN (4, 10, 12, 25, 43, 44, 45, 46, 47, 51, 52, 55, 61, 62, 66, 67, 68, 70, 75, 77, 79, 80, 81, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109)
	GROUP BY t1.skc
	HAVING totalontheway > 0
) t3;
```

sql太复杂了,业务逻辑处理吧

![image-20210108110303278](/Users/lumac/Library/Application Support/typora-user-images/image-20210108110303278.png)

## ae23dd26608942c7c2f51d5284dbde79 已解决

```sql
update dms_goods_extra set is_delete=1 where update_time < '2021-01-08' and is_delete = 0 limit 1000;
```

全表更新了,先查出最小的id,然后分批处理

## c7c3b7160506b0820dbfbde3e10dde4f 已解决

频率:159次

耗时:1-2.05

```sql
select id, vhost, exchange, queue_name, fail_times, `status`, is_delete, add_uid, add_time, update_uid, update_time, review_uid, review_time, flag, bean_name, business_id, exception_type, times_tamp, application_type , message from dms_log_mq where ( is_delete = 0 and `status` = 1 and queue_name in ( 'dms_product_productionteam_queue' , 'dms_cost_consume' ) );
```

![image-20210112171207889](/Users/lumac/Library/Application Support/typora-user-images/image-20210112171207889.png)

status加索引

## 3cd76d57cb0d7bfe9967f02f1b0d3316 已解决

使用场景:

耗时:1.2-1.4

```sql
select id, skc, category_id, sales_volume, date_format(s_added_time,'%y-%m-%d') as saddedtime, date_format(r_added_time,'%y-%m-%d') as raddedtime from `dms_goods` where is_delete = 0 and s_sale_status = 1 and sales_volume > 0 and date(s_added_time) <= '2021-01-10' and s_added_time > '1970-01-02' and category_id in ( 2048 , 2177 , 2178 , 1925 , 2053 , 1926 , 2054 , 1936 , 1937 , 2322 , 2706 , 1938 , 2323 , 1939 , 2708 , 2324 , 2325 , 2326 , 1942 , 2327 , 2328 , 2329 , 2330 , 2586 , 2331 , 2587 , 2332 , 2333 , 2463 , 2464 , 2465 , 2466 , 2467 , 2215 , 2216 , 2217 , 2218 , 2219 , 2220 , 2221 , 1966 , 2222 , 2223 , 2224 , 2225 , 2226 , 2227 , 2228 , 2229 , 2230 , 2231 , 2232 , 2233 , 2234 , 2235 , 2236 , 2237 , 2238 , 2239 , 2240 , 1888 , 1889 , 2017 , 1890 , 1891 , 1892 , 1893 , 2280 , 2281 , 2668 ) and maintenance_uid in ( '生产维护三组' , '义乌饰品组' , 'al儿童鞋子组' , '市场男装十三行组' , '沙河一组' , '样衣' , 'al内衣组' , '外协牛仔组' , '生产童装一组' , 'al男士非成衣组' , '品牌vmi组' , 'al泳衣组' , 'al饰品组' , 'al数码组' , '生产童装二组' , 'al儿童非成衣组' , 'rw二组' , '沙河二组' , '市场男装阿里组' , '沙河内衣组' , '生产premium组' , '生产维护四组' , '市场印度成衣组' , '市场大码十三行组' , 'al女包组' , '市场童装阿里组' , '生产motf组' , '市场成衣订单组' , '市场家居南通组' , '十三行一组' , '市场男装沙河组' , '美国' , '市场中东成衣组' , 'er单独款' , '生产维护八组' , '市场礼服组' , '生产维护二组' , 'al女士首饰组' , '十三行内衣组' , '市场大码订单组' , '市场五组' , 'al女士饰品组' , '生产毛衣组' , '中山八童装' , '市场童装浙江组' , '外协大码组' , '生产维护一组' , '外协二组' , '市场泳衣二组' , 'al彩妆组' , '外协童装组' , 'al成衣一组' , '品牌组' , '市场大码常熟组' , '市场女鞋一组' , '生产维护五组' , '外协一组' , '美国市场款鞋子' , '美国鞋子' , 'al成衣二组' , '流花男装组' , '亚马逊成衣组' , 'al女士鞋子组' , '生产维护七组' , '生产童装组' , '生产中东组' , '生产维护十组' , '市场大码阿里组' , 'al睡衣组' , '生产男装组' , '外协三组' , '生产孕妇装' , '常熟一组' , '欧洲' , '生产童装三组' , '市场童装' , '沙河三组' , 'al订单组' , '生产维护九组' , '市场家居阿里组' , '市场中东非成衣组' , '外协内衣组' , '品牌vmi非成衣组' , '市场大码沙河组' , '生产大码组' , 'vmi' , '外协男装组' , '市场大码礼服组' , 'rw一组' , 'al男士鞋子组' , 'al女士配饰组' , '市场家居义乌组' , '生产维护六组' , '市场孕妇装' ) and goods_level_id in ( 86 , 87 , 55 , 61 , 62 , 81 , 75 , 88 , 25 , 89 , 43 , 12 , 42 , 78 , 33 , 10 , 47 , 17 , 45 , 46 , 77 , 68 , 70 , 84 , 96 , 53 , 51 , 97 , 58 , 105 , 95 , 91 , 92 , 72 , 66 , 94 , 90 , 80 , 104 , 85 , 101 , 79 , 4 , 93 , 65 , 76 , 67 , 34 , 52 , 54 , 69 , 83 , 64 , 59 , 6 , 32 , 8 , 60 , 44 , 7 , 39 , 15 , 100 , 102 , 103 , 106 ) order by id asc;
```

![image-20210111174948881](/Users/lumac/Library/Application Support/typora-user-images/image-20210111174948881.png)

category_id添加索引后执行计划:效果显著

![image-20210111175339418](/Users/lumac/Library/Application Support/typora-user-images/image-20210111175339418.png)

## 9d70f6bea89c3d73ead44dfdfe053a80

使用场景:vmi

```sql
select h.generate_time, h.skc, h.goods_level_id, h.auto_add_order, s.title suppliername, dsc.title suppliercategoryname, vs.size, vs.sales_volume_7, vs.to_send_goods_num, vs.ontheway_num, vs.stock_guang_zhou, vs.wait_ontheshelf, vs.sale_days, vs.predict_sales, vs.stock_vigilance, vs.spot_advice_calculate, vs.add_advice_calculate, vs.spot_advice, vs.add_advice, vs.auto_add_order_num, vs.add_order_num from dms_vmi_stock_history h left join dms_vmi_stock_history_size vs on vs.history_id=h.id left join dms_supplier s on s.id=h.supplier_id join dms_supplier_category dsc on s.type = dsc.type and s.category_id = dsc.srm_supplier_category_id where h.is_delete=0 and s.is_delete=0 and h.generate_time >='2020-12-27 00:00:00' and h.generate_time <='2021-01-11 23:59:59' order by h.generate_time desc limit 0,50000;
```

数据太多了,按天分批取,最后汇总,开个多线程.

## ae23dd26608942c7c2f51d5284dbde79 已解决

使用场景:分批删除非当日更新的数据

频率:8次

```sql
update dms_goods_extra set is_delete=1 where update_time < '2021-01-11' and is_delete = 0 limit 1000;
```

```sql
select id from  dms_goods_extra  where update_time < '2021-01-11' and is_delete = 0 order by id asc limit 1;
```

先取出最小id,然后通过id更新,效果不错哦.

## c5fae01841b01de88df2cf684053721a

使用场景:getAgingByOrder

频率:123+次

```sql
select count(*) from dms_order where is_delete =0 and order_status_value in (1,2,3,4,5,6) and add_time >= '2020-12-14' and add_time <= '2020-12-21' and channel_id=1 and urgency_value in ( 1 );
```

![image-20210112171331010](/Users/lumac/Library/Application Support/typora-user-images/image-20210112171331010.png)

![image-20210112172038224](/Users/lumac/Library/Application Support/typora-user-images/image-20210112172038224.png)

## 8b5a08dcf2f74d7678741ac2f9b90052 1.12号上线

```sql
select dg.id, dg.product_time, show_status, dg.skc, s_sale_status, r_sale_status, s_sale_by_inventory, r_sale_by_inventory, s_added_time, r_added_time, s_gross_profit_margin, r_gross_profit_margin, goods_thumb, average_return_time, goods_level_id, dg.maintenance_uid, short_in_skc_status, dg.short_in_size_status, dg.warn_inventory_status, delivery_time, supplier_id, e.buyer, supplier_code, production_team_id, production_team_name, season_id, outofstock_total, first_order_date, stock_date, bi_effect_date, dg.add_time, dg.update_time, dg.add_uid, dg.update_uid, dg.is_delete, dg.last_update_time, dg.first_order_id, order_count, deposit_sale_ratio, sales_volume, s_sales_volume, r_sales_volume, return_rate_30_days, return_rate_60_days, s_added_time, r_added_time, dg.real_time_sales_volume, e.target_inventory, e.demand_inventory, dg.eval_score, dg.material_status, ds.type, dg.printing from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id left join dms_goods_expand e on e.skc=dg.skc and e.is_delete=0 where dg.is_delete = 0 and dg.show_status = 2 and dg.maintenance_uid in ( 'rw一组' , 'rw二组' ) and dg.goods_level_id in ( 10 ) and dg.s_sale_status = 1 order by dg.sales_volume desc, dg.id desc limit 0,50;
```

优化:加一个skc排序

```sql
select dg.id, dg.product_time, show_status, dg.skc, s_sale_status, r_sale_status, s_sale_by_inventory, r_sale_by_inventory, s_added_time, r_added_time, s_gross_profit_margin, r_gross_profit_margin, goods_thumb, average_return_time, goods_level_id, dg.maintenance_uid, short_in_skc_status, dg.short_in_size_status, dg.warn_inventory_status, delivery_time, supplier_id, e.buyer, supplier_code, production_team_id, production_team_name, season_id, outofstock_total, first_order_date, stock_date, bi_effect_date, dg.add_time, dg.update_time, dg.add_uid, dg.update_uid, dg.is_delete, dg.last_update_time, dg.first_order_id, order_count, deposit_sale_ratio, sales_volume, s_sales_volume, r_sales_volume, return_rate_30_days, return_rate_60_days, s_added_time, r_added_time, dg.real_time_sales_volume, e.target_inventory, e.demand_inventory, dg.eval_score, dg.material_status, ds.type, dg.printing from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id left join dms_goods_expand e on e.skc=dg.skc and e.is_delete=0 where dg.is_delete = 0 and dg.show_status = 2 and dg.maintenance_uid in ( 'rw一组' , 'rw二组' ) and dg.goods_level_id in ( 10 ) and dg.s_sale_status = 1 order by dg.sales_volume desc, dg.id,dg.skc desc limit 0,50;
```

## 0a274429ad451ec6df430816f3436a61 传入ids,今天上线,已解决

```sql
SELECT t1.id, t1.skc, t1.sales_volume
	, date_format(t1.s_added_time, '%y-%m-%d') AS saddedtime
	, date_format(t1.r_added_time, '%y-%m-%d') AS raddedtime
FROM dms_goods t1
WHERE t1.supplier_id IN (
		SELECT DISTINCT supplier_id
		FROM dms_supplier_label_rel
		WHERE label_code = 'vmi_supplier'
			AND label_status = 1
			AND is_delete = 2
	)
	AND t1.id > 1519870
ORDER BY t1.id ASC,t1.skc
LIMIT 10000;
```

## 6c29e9766eb5512df8c7fdd40009d54e 疑难

耗时:3466ms

使用场景:新商品层次查询数据

频率:69次

```sql
select dg.id,
       dg.skc,
       dg.sales_volume,
       dg.s_gross_profit_margin,
       ds.type,
       dg.category_id,
       dg.goods_level_id,
       dg.maintenance_uid,
       date_format(dg.s_added_time, '%y-%m-%d') as saddedtime,
       date_format(dg.r_added_time, '%y-%m-%d') as raddedtime
from `dms_goods` dg
         left join dms_supplier ds on dg.supplier_id = ds.id
where dg.is_delete = 0
  and dg.s_sale_status = 1
  and dg.sales_volume > 0
  and date(dg.s_added_time) <= '2021-01-10'
  and dg.s_added_time > '1970-01-02'
  and dg.maintenance_uid in
      ('生产维护三组', '义乌饰品组', 'al儿童鞋子组', '市场男装十三行组', '沙河一组', '样衣', 'al内衣组', '外协牛仔组', '生产童装一组', 'al男士非成衣组', '品牌vmi组',
       'al泳衣组', 'al饰品组', 'al数码组', '生产童装二组', 'al儿童非成衣组', 'rw二组', '沙河二组', '市场男装阿里组', '沙河内衣组', '生产premium组', '生产维护四组',
       '市场印度成衣组', '市场大码十三行组', 'al女包组', '市场童装阿里组', '生产motf组', '市场成衣订单组', '市场家居南通组', '十三行一组', '市场男装沙河组', '美国', '市场中东成衣组',
       'er单独款', '生产维护八组', '市场礼服组', '生产维护二组', 'al女士首饰组', '十三行内衣组', '市场大码订单组', '市场五组', 'al女士饰品组', '生产毛衣组', '中山八童装',
       '市场童装浙江组', '外协大码组', '生产维护一组', '外协二组', '市场泳衣二组', 'al彩妆组', '外协童装组', 'al成衣一组', '品牌组', '市场大码常熟组', '市场女鞋一组', '生产维护五组',
       '外协一组', '美国市场款鞋子', '美国鞋子', 'al成衣二组', '流花男装组', '亚马逊成衣组', 'al女士鞋子组', '生产维护七组', '生产童装组', '生产中东组', '生产维护十组',
       '市场大码阿里组', 'al睡衣组', '生产男装组', '外协三组', '生产孕妇装', '常熟一组', '欧洲', '生产童装三组', '市场童装', '沙河三组', 'al订单组', '生产维护九组',
       '市场家居阿里组', '市场中东非成衣组', '外协内衣组', '品牌vmi非成衣组', '市场大码沙河组', '生产大码组', 'vmi', '外协男装组', '市场大码礼服组', 'rw一组', 'al男士鞋子组',
       'al女士配饰组', '市场家居义乌组', '生产维护六组', '市场孕妇装')
  and dg.goods_level_id in
      (86, 87, 55, 61, 62, 81, 75, 88, 25, 89, 43, 12, 42, 78, 33, 10, 47, 17, 45, 46, 77, 68, 70, 84, 96, 53, 51, 97,
       58, 105, 95, 91, 92, 72, 66, 94, 90, 80, 104, 85, 101, 79, 4, 93, 65, 76, 67, 34, 52, 54, 69, 83, 64, 59, 6, 32,
       8, 60, 44, 7, 39, 15, 100, 102, 103, 106)
limit 455000, 5000;
```

![image-20210112111058290](/Users/lumac/Library/Application Support/typora-user-images/image-20210112111058290.png)

## 44148bf7e3bdfc84947422d719f55384 类似 已解决

和c7c3b7160506b0820dbfbde3e10dde4f雷同

10次

## 48a1efbe5c1a5c3e40a0642348377cb3 类似 已解决

和c7c3b7160506b0820dbfbde3e10dde4f雷同

3次

## e0b50721457b4ea1a262f7f3d1f764fd

```sql
select max(id) as maxcursorkey, min(id) as mincursorkey from (select id from dms_goods where is_delete = 0 and show_status = 2 limit 352974, 176487) t;
```

```sql
select id 
FROM (
	SELECT id
	FROM dms_goods
	WHERE is_delete = 0
		AND show_status = 2
	LIMIT 352974, 176487
) a order by a.id desc  limit 1;
select id 
FROM (
	SELECT id
	FROM dms_goods
	WHERE is_delete = 0
		AND show_status = 2
	LIMIT 157679, 157679
) a order by a.id asc  limit 1;
```

## baea806409c2544832f22ef6d1118b35 疑难

频率:40次

耗时:1-3.6

```sql
select s.id as id, s.supplier_id as supplierid, s.skc as skc, s.goods_type as goodstype, e.id as eid, e.jit_available_num as jitavailablenum, e.require_delivery_num as requiredeliverynum, e.size as size, s.skc as eskc from dms_jit_stock s left join dms_jit_stock_extend e on e.jit_id = s.id left join dms_goods g on g.skc = s.skc left join dms_supplier si on si.id = g.supplier_id where s.is_delete = 0 and e.is_delete = 0 and e.jit_available_num > 0 and g.sales_volume > 0 and si.type = 1 order by skc limit 115000,5000;
```

## 807dc7b8e0163a5e8abb76847972390f 已解决

耗时:1.24-1.3

频率:8次

使用场景:count(dsp.id)

```sql
SELECT COUNT(dsp.id) AS stock_param_group_count, dsp.id, dsp.skc, dsp.dimensionality
	, dsp.is_del, dsp.size, dsp.change_source, dsp.goods_time, dsp.stock_up_day
	, dsp.min_order, dsp.sell_coefficient, dsp.spot_goods_time
FROM dms_stock_param dsp
	JOIN dms_goods dg ON dsp.skc = dg.skc
WHERE dsp.is_del = 2
	AND dsp.goods_time >= 3
	AND dsp.goods_time <= 9
	AND dg.maintenance_uid IN (
		'市场大码订单组', 
		'al男士非成衣组', 
		'al饰品组', 
		'al女士鞋子组', 
		'市场女鞋一组', 
    
		'al数码组', 
		'市场成衣订单组', 
		'al订单组', 
		'al儿童非成衣组', 
		'市场家居阿里组'
	)
	AND dg.goods_level_id IN (80, 84, 87, 88, 90, 10, 109, 61, 47)
GROUP BY dsp.skc
ORDER BY dsp.skc, dsp.id
LIMIT 10000;
```

dms_stock_param没有走索引

## ![image-20210112185054251](/Users/lumac/Library/Application Support/typora-user-images/image-20210112185054251.png)9c94d54910f32b6e9134c25abba01d94  已分配 已解决

```sql
SELECT add_time, storage_time
FROM dms_sub_order
WHERE is_delete = 0
	AND show_flag = 0
	AND add_time >= '2021-01-04'
	AND add_time <= '2021-01-11'
	AND channel_id = 2
	AND urgency_value IN (2, 3)
LIMIT 0, 1000;
```

![image-20210113101310769](/Users/lumac/Library/Application Support/typora-user-images/image-20210113101310769.png)

优化方案:

1.先取出最小id

```sql
SELECT min(id)
FROM dms_sub_order
WHERE is_delete = 0
	AND show_flag = 0
	AND add_time >= '2021-01-04'
	AND add_time <= '2021-01-11'
	AND channel_id = 2
	AND urgency_value IN (2, 3)
```

2.一批一批的取

```sql
    SELECT add_time, storage_time,id
FROM dms_sub_order
WHERE is_delete = 0
	AND show_flag = 0
	AND add_time >= '2021-01-04'
	AND add_time <= '2021-01-11'
	AND channel_id = 2
	AND urgency_value IN (2, 3)
    AND id > 237607598
    order by id desc
LIMIT 1000;
```

## 1741512bfe47f300baf14f3a75df1001 已解决

场景: 统计当日建议数

```sql
select count(1) from dms_maintenance_advice a left join dms_goods_maintenance m on m.id=a.maintenance_id where a.is_delete=0 and m.maintenance_uid='沙河一组' and date_format(a.last_update_time,'%y-%m-%d') =date_format('2021-01-13 11:15:08.689','%y-%m-%d');
```

优化建议:

date传参,逻辑放到sql外

## 1a6741e8071028c94fd5617e37684d7c 代码 已上线 已解决

备货标签页面搜索

解决方案:代码中排

```sql
select ds.id, ds.skc, dg.title labeltitle, dgsl.title labeltypetitle from dms_skc_rel_label ds left join dms_goods_stock_label dg on dg.id =ds.label_id left join dms_goods_stock_label dgsl on dgsl.id=ds.label_type_id where ds.is_del=0 order by ds.label_type_id,ds.label_id,ds.id desc limit 0,50;
```

![image-20210114111604925](/Users/lumac/Library/Application Support/typora-user-images/image-20210114111604925.png)

## 88051d26783a78aa46660d16f6509977 已解决

```sql
select count(*) from dms_sub_order where is_delete =0 and show_flag = 0 and add_time >= '2020-12-28' and add_time <= '2021-01-04' and channel_id=1 and urgency_value in ( 1 );
```

根据addtime分批count汇总



## 34ec814b02a5c9ae73da5b586dbf2f8d 已解决

```sql
update dms_inventory_warning set is_delete = 1 where type = 1 ;
```

使用场景:清理上一次今日统计

优化type is_delete加索引,sql改成

```sql
update dms_inventory_warning set is_delete = 1 where type = 1 and is_delete = 0;
```

## fe6aae23e69017ab7cf821fce0cda540 已解决

使用场景:获取子单信息

```sql
SELECT *
FROM (
	SELECT do.order_id, do.sub_order_id AS suborderid, dos.id AS sizeprimaryid, dos.wait_ontheshelf AS waitontheshelf, dos.has_sent_to_scnum AS hassenttoscnum
		, cancel_reason AS cancelreasonvalue, do.completion_time AS completiontime, do.package_time AS packagetime, do.skc, dos.size
		, dos.order_amount, dos.delivery_amount, dos.receipt_amount, dos.storage_amount, dos.defective_amount
		, dos.cut_amount, dos.price, do.currency_id, do.add_uid, do.review_uid
		, date_format(do.predict_receipt_date, '%y-%m-%d %h:%i:%s') AS predict_receipt_date
		, date_format(do.demand_receipt_date, '%y-%m-%d %h:%i:%s') AS demand_receipt_date
		, date_format(do.add_time, '%y-%m-%d %h:%i:%s') AS add_time
		, date_format(do.delivery_time, '%y-%m-%d %h:%i:%s') AS delivery_time
		, date_format(do.receipt_time, '%y-%m-%d %h:%i:%s') AS receipt_time
		, date_format(do.storage_time, '%y-%m-%d %h:%i:%s') AS storage_time
		, date_format(do.refused_time, '%y-%m-%d %h:%i:%s') AS refusedtime
		, date_format(do.offshelf_time, '%y-%m-%d %h:%i:%s') AS offshelftime
		, date_format(do.obsolete_time, '%y-%m-%d %h:%i:%s') AS obsoletetime, do.system_flag
		, do.order_status_value, do.supplier_id, do.first_mark_value, do.prepare_type_value, do.order_mark_value
		, do.urgency_value, do.fba_account, do.is_production_completion, do.delivery_tag, do.delivery_tag_uid
		, date_format(do.delivery_tag_time, '%y-%m-%d %h:%i:%s') AS delivery_tag_time, do.is_end_year_first_order
		, date_format(do.demand_finish_production_time, '%y-%m-%d %h:%i:%s') AS demandfinishproductiontime, do.production_team_id AS productionteamid
		, do.maintenance_name AS maintenanceuid, do.category_value AS categoryvalue, do.remark
	FROM dms_sub_order do
		LEFT JOIN dms_sub_order_size dos
		ON do.sub_order_id = dos.sub_order_id
			AND dos.order_amount > 0
		LEFT JOIN dms_supplier ds ON do.supplier_id = ds.id
	WHERE do.is_delete = 0
		AND do.add_time >= '2021-01-05 00:00:00'
		AND do.add_time <= '2021-01-05 23:59:59'
		AND do.show_flag = 0
		AND do.order_status_value IN (8, 20, 21, 22, 23)
) uniontable
ORDER BY add_time DESC, suborderid DESC, order_id DESC, sizeprimaryid ASC
```

这个排序是一定需要的吗?

## fd2066d655f00e3e72c92af2370250f8 已解决

```sql
select * from dms_sold_out_goods where is_delete = 0 and supplier_id = 1001539;
```

索引缺失

添加supplier_id索引

## edd62f6d883ad12a150ecee4a1e258dc 已解决

join内部不走索引

supplier_id太多了

```sql
select count(1) from dms_goods dg join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc where dg.show_status = 2 and dg.goods_level_id in ( 10 , 61 ) and dg.supplier_id in ( 1223 , 8902 , 8923 , 5417 , 8063 , 5395 , 9012 , 7641 , 1988 , 1491 , 7640 , 5416 , 9700 , 7073 , 5953 , 10447 , 9742 , 9746 , 5988 , 8775 , 6941 , 8062 , 8181 , 8881 , 5418 , 8886 , 9877 , 5361 , 9420 , 9743 , 5636 , 5797 , 1528 , 9013 , 9750 , 6025 , 8883 , 8333 , 5456 , 1539 , 5553 , 9116 , 5917 , 9115 , 8889 , 1493 , 5432 , 1562 , 5455 , 1837 , 9015 , 1993 , 8882 , 8202 , 9010 , 6236 , 9182 , 8884 , 6821 , 8885 , 9745 , 10881 , 1579 , 7394 , 9789 , 1498 , 7138 , 7364 , 7289 , 1497 , 7072 , 1994 , 2070 , 7523 , 7766 , 5472 , 8234 , 7498 , 1496 , 1849 , 7384 , 7767 , 1728 , 5634 , 1727 , 7392 , 1992 , 5420 , 5918 , 6942 , 1503 , 8152 , 5552 , 5458 , 8201 , 8035 , 7666 , 6910 , 1499 , 5633 , 1583 , 6936 , 5554 , 7395 , 7991 , 7529 , 5796 , 1713 , 5635 , 1586 , 8192 , 7249 , 8179 , 8180 , 7526 , 7927 , 5550 , 7765 , 9252 , 28586 , 1000530 , 1000057 , 1000320 , 1000947 , 1000047 , 1000029 , 10883 , 1000323 , 1000868 , 1000822 , 1000091 , 27750 , 1000901 , 29899 , 29842 , 1001151 , 28457 , 1001547 , 29829 , 1001077 , 28683 , 1001225 , 9491 , 1000654 , 29991 , 1001587 , 1000039 , 29105 , 27948 , 1000024 , 1000798 , 1000524 , 28507 , 1000812 , 1000329 , 1000022 , 1000442 , 1001305 , 1000357 , 1000509 , 1000985 , 1000282 , 1001139 , 1000668 , 1000054 , 1000459 , 1000019 , 1000278 , 1000777 , 1000017 , 29514 , 10576 , 1000088 , 28737 , 8946 , 29498 , 1000756 , 1001142 , 1000614 , 28604 , 1001075 , 1000055 , 29417 , 1000375 , 1000421 , 28588 , 28682 , 1000933 , 9519 , 1000531 , 1000372 , 1001299 , 28639 , 7199 , 1000942 , 1000787 , 1000847 , 1000912 , 1000200 , 1000092 , 28058 , 1000657 , 1000226 , 1000535 , 10448 , 1001315 , 1000973 , 1000612 , 28802 , 1000056 , 1000307 , 27909 , 1001072 , 27908 , 1000295 , 1000049 , 1000061 , 1001114 , 1000105 , 1001427 , 1000044 , 1000249 , 1001536 , 1001452 , 1000290 , 1000294 , 1000028 , 1001431 , 1000503 , 28458 , 1000422 , 1000620 , 1001300 , 1000807 , 1001321 , 28528 , 1001061 , 9744 , 1000281 , 1001303 , 1000532 , 10222 , 7943 , 1000533 , 1000209 , 1000058 , 29678 , 1000093 , 29910 , 1000062 , 27671 , 29609 , 28589 , 28543 , 1000982 , 1000051 , 1000857 , 1001359 , 1000902 , 10882 , 9517 , 29687 , 1000538 , 1500 , 1000761 , 1000043 , 30352 , 1000449 , 1000462 , 29717 , 29916 , 29867 , 1000710 , 1001414 , 1001314 , 27679 , 27805 , 30333 , 1000888 , 1000279 , 1000031 , 1001289 , 1000454 , 8879 , 1000563 , 1000455 , 1000259 , 1001162 , 1001510 , 1000218 , 1000038 , 1001134 , 1001532 , 5954 , 28019 , 1000274 , 8766 , 1000611 , 1000334 , 27758 , 1000929 , 28749 , 8880 , 28412 , 1000828 , 1001323 , 10864 , 1000310 , 1000457 , 29316 , 1000015 , 1000484 , 28719 , 8508 , 1000967 , 1000913 , 28756 , 1000974 , 1000033 , 1000318 , 1501 , 1000850 , 1000759 , 28554 , 1000046 , 1000445 , 1000280 , 1000899 , 10575 , 7928 , 27767 , 1000309 , 1001485 , 1000365 , 1000265 , 9421 , 1001105 , 1000549 , 9011 , 1000053 , 1001413 , 30367 , 1000706 , 28029 , 1000504 , 1001538 , 1000927 , 2130 , 1000040 , 27982 , 27709 , 1000808 , 29941 , 1000331 , 1000476 , 1001328 , 30314 , 28811 , 1729 , 28818 , 1001573 , 1000585 , 1000014 , 28602 , 1001237 , 1001178 , 9117 , 1000371 , 1000619 , 1001308 , 1000333 , 1001471 , 1000903 , 29967 , 28513 , 1000305 , 28679 , 1000809 , 1000287 , 1000304 , 1000232 , 1001254 , 1000030 , 1001147 , 1001076 , 1000914 , 1000275 , 28548 , 29942 , 1000693 , 1001017 , 1000437 , 1000027 , 1000491 , 1001380 , 29896 , 1000348 , 28748 , 1001310 , 1001412 , 1000930 , 1000099 , 1000381 , 29830 , 1001515 , 1000904 , 1000702 , 10152 , 1000308 , 1000670 , 27993 , 1000478 , 28684 , 1000045 , 1001230 , 1000119 , 1000052 , 1000376 , 1000969 , 1000020 , 1000780 , 28796 , 27918 , 28617 , 1000990 , 6937 , 1000965 , 1000950 , 1000872 , 1001405 , 1000324 , 28610 , 1001073 , 1000498 , 1000917 , 27981 , 27633 , 1001152 , 1000935 , 1000332 , 1001185 , 10511 , 1000390 , 10884 , 1001475 , 1001234 , 10153 , 1000059 , 1000023 , 10223 , 27864 , 1001097 , 29496 , 29836 , 1000420 , 1000048 , 27664 , 5987 , 29798 , 1000035 , 9490 , 29840 , 1001322 , 1000298 , 1000060 , 1001122 , 1001319 , 1000283 , 1000785 , 27920 , 28516 , 10885 , 1001282 , 1001078 , 1488 , 27921 , 1000645 , 1001044 , 1000971 , 1001277 , 1000471 , 1000697 , 1000750 , 28692 , 1001373 , 1000500 , 1000089 , 29839 , 1001468 , 29351 , 10891 , 1001120 , 1000103 , 28508 , 1000042 , 1000223 , 1000032 , 1000796 , 29841 , 1000016 , 1000368 , 28741 , 1000041 , 1001010 , 1000817 , 1000102 , 1001473 , 29898 , 1000026 , 1000916 , 1000873 , 29887 , 1000328 , 1000534 , 1000277 , 1001540 , 1000848 , 10858 , 10896 , 10574 , 1001353 , 1001118 , 1000025 , 1000306 , 28907 , 1000680 , 27736 , 10849 , 1000034 , 27823 , 1001345 , 28587 , 1001255 , 1001032 , 28407 , 1000021 , 27807 , 1001054 , 1000198 , 1000090 , 29831 , 1000292 , 6847 , 27438 , 1000894 , 1000811 , 1001428 , 2089 , 1000542 , 29699 , 1001084 , 1001199 , 1001273 , 1001156 , 1000036 , 1001482 , 1000966 , 1000695 , 30116 , 28527 , 29643 , 1000839 , 9203 , 28532 , 1000273 , 1000342 , 1000082 , 1000018 , 28736 , 28492 , 1001556 , 29485 , 29611 , 1000069 , 1000296 , 28591 , 1000355 , 1000395 , 1000087 , 1001052 , 7522 , 9183 , 9694 , 9143 , 1000475 , 1000037 , 1000738 , 1000086 , 1001262 , 1000448 );
```

## bbc36c62d344110b00133ea7b59f3151 已解决



## a9fe92aedbb7f0c92dd146a1d7729d15 已加索引 已解决

```sql
update dms_sales_rate_analyze set is_delete = 1 where is_delete = 0;
```

## 933cbc009237b3f78886d17ca185cd34 已解决

```sql
select id, analysis_id, sales_volume, skc_num, sales_volume_rate, skc_rate, rate, sales_threshold, country, add_time_start, add_time_end, sales_volume_total, skc_total, add_time, update_time, add_uid, update_uid, is_delete, last_update_time, this_week_sales_threshold, last_week_sales_threshold, add_days from dms_sales_rate_analyze where is_delete=0 and add_days = 1 and country = 21 and is_delete = 0;
```

## 50d0f788aa5dd2caddd8ea8520569174 已优化 待上线

分批获取warehouse信息

## b1612ba92176d247889b44aa1c099118 疑难

```sql
select dso.sub_order_id as suborderid, dso.order_id as orderid, dg.maintenance_uid as maintenanceuid, dg.goods_level_id as goodslevelid, dso.system_flag as systemflag, dso.demand_finish_production_time, dso.delivery_tag_time from dms_sub_order dso left join dms_goods dg on dso.skc = dg.skc where dso.is_delete = 0 and dso.order_status_value in ( 23 , 22 , 21 , 20 , 8 , 7 ) and dso.order_mark_value not in ( '812' , '804' , '816' , '831' , '736' ) and dso.prepare_type_value = '9' and dso.system_flag = 1 and dso.delivery_tag != 1 and not exists ( select 1 from dms_exception_order deo where deo.exception_type = 12 and deo.type = 2 and find_in_set('4', deo.exception_form) and deo.status = 1 and deo.is_delete = 0 and deo.sub_order_id = dso.sub_order_id ) limit 55000,5000;
```

![image-20210119164035372](/Users/lumac/Library/Application Support/typora-user-images/image-20210119164035372.png)

## a2a60e68b431d315a085fa4060e49ed4

分批统计

```sql
select count(1) from dms_goods dg where dg.show_status = 2 and dg.maintenance_uid in ( '市场男装阿里组' , '市场大码阿里组' , 'al内衣组' , '市场童装阿里组' , 'al成衣二组' , 'al成衣一组' , 'al睡衣组' , 'al泳衣组' , '市场中东成衣组' , '市场家居南通组' , '常熟一组' , '市场大码十三行组' , '十三行一组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '生产毛衣组' , '生产男装组' , '生产童装组' , '生产中东组' , '生产大码组' , '生产维护六组' , '生产维护五组' , '生产维护四组' , '生产维护三组' , '生产维护二组' , '生产维护一组' , '品牌组' , '外协牛仔组' , '外协内衣组' , '外协男装组' , '外协大码组' , '外协童装组' , '外协二组' , '外协一组' , '美国' , '欧洲' , '流花男装组' , '样衣' , '市场男装十三行组' , '生产维护八组' , '生产premium组' , '生产孕妇装' , '品牌vmi组' , '生产维护七组' , '生产童装一组' , '生产童装二组' , '市场孕妇装' , '美国市场款鞋子' , '美国鞋子' , '生产维护九组' , '外协三组' , '生产维护十组' , '生产童装三组' , '市场泳衣二组' , '生产motf组' , '市场礼服组' );
```

