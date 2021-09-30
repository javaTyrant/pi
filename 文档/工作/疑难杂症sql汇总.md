## 1.8b5a08dcf2f74d7678741ac2f9b90052

```sql
select dg.id from dms_goods dg left join dms_supplier ds on dg.supplier_id = ds.id left join dms_goods_expand e on e.skc=dg.skc and e.is_delete=0 join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 1 and label_type_status = 2 and label_status = 2 and label_id in ( 9 , 11 , 14 ) ) r1 on dg.skc = r1.skc join ( select distinct skc from dms_skc_rel_label where is_del = 0 and label_type_id = 39 and label_type_status = 2 and label_status = 2 and label_id in ( 40 ) ) r39 on dg.skc = r39.skc where dg.is_delete = 0 and dg.show_status = 2 and dg.maintenance_uid in ( '市场男装沙河组' ) and dg.sales_volume >= '50' and dg.goods_level_id in ( 10 , 61 ) and dg.deposit_sale_ratio_total <= '7.5' and dg.maintenance_uid in ( '市场中东成衣组' , '市场大码沙河组' , '市场男装沙河组' , '沙河内衣组' , '沙河三组' , '沙河二组' , '沙河一组' , '流花男装组' , '样衣' ) order by dg.sales_volume desc, dg.id desc limit 0,50;
```

## 2.