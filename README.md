# SAP format dataset
## Schema
### T002
- SPRAS primary key
### T005
- LAND1 primary key
- XEGLD
### KNA1
- KUNNR primary key
- LAND1 references T005
- NAME1
- SORTL
- ERDAT
- ERNAM
### VBAK
- VBELN primary key
- ERDAT
- ERZET
- ERNAM
- AUDAT
- AUART
- VKORG
- KUNNR references KNA1
### MARA
- MATNR primary key
- ERSDA
- ERNAM
- LAEDA
- AENAM
- LVORM
- MTART
- MEINS
- URL
### MAKT
- MATNR references MARA
- SPRAS references T002
- MAKTX
- primary key(MATNR, SPRAS)
### VBAP
- VBELN references VBAK
- POSNR
- MATNR references MARA
- MEINS
- primary key(VBELN, POSNR)
## Table contents
### T002
    EN
### T005
    CH|
    DE|X
    FR|X
    IT|X
    UK|X
    US|
### KNA1
    0000500000|IT|Guccion Gucci S.p.A|gucci50123|20100201|sandrag
    0000500001|IT|Prada S.p.A|prada20135|20110303|accardia
    0000500002|UK|River Island Clothing co. Limited|rive636095|20100305|smithds
    0000500003|UK|Boohoo Group plc|boohoo7297|20100115|sandrag
### VBAK
    0015000000|20100215|15:15:01|sandrag|20100215|OR|SDNL|0000500001
    0015000001|20100216|12:14:44|sandrag|20100216|OR|SDNL|0000500002
### MARA
    000000000400000000|20100901|gallage|20180106|simonr||BAG1|EA|https://upload.wikimedia.org/wikipedia/commons/d/d5/Laptop_bags_luxury_diManolo_%2812_of_15%29.jpg
    000000000400000001|20100901|gallage|20190116|simonr||BAG1|EA|https://pxhere.com/en/photo/1430309
### MAKT
    000000000400000000|EN|Manolo Blahnik bag women
    000000000400000001|EN|Cartera handbag
### VBAP
    0015000000|000010|000000000400000000|EA
    0015000000|000020|000000000400000001|EA
    0015000001|000010|000000000400000001|EA
## Download and decompress Olist dataset
[Brazilian E-Commerce Public Dataset by Olist, version 7](https://www.kaggle.com/olistbr/brazilian-ecommerce/)
## Create and execute SQLite initialization script from this README.md
    DROP TABLE if exists readme;
    CREATE TABLE readme(line);
    .separator \t
    .import README.md readme
    .headers off
    .once initdb.sqlite
    select substr(line, 5) from readme where rowid > (select rowid from readme where line like '### Create and%' limit 1);
    .read initdb.sqlite
### Create and populate SAP tables
    PRAGMA foreign_keys = ON;
    CREATE VIEW if not exists VREAD(row, line) as select rowid, line from readme;
    CREATE VIEW if not exists VDATA as select * from vread
      where row between
        (select row+1 from vread where line like '%## Schema%' limit 1) and
        (select row-1 from vread where line like '%## Download%' limit 1);
    CREATE VIEW if not exists VSQL as select
      case substr(vdata.line, 1, 4)
        when '### ' then
          iif(substr(vdata_next.line, 1, 2) = '- ',
            'CREATE TABLE if not exists' || substr(vdata.line, 4) || '(' ,
            'REPLACE INTO' || substr(vdata.line, 4) || ' values ')
        when '## T' then ''
        when '    ' then
          iif(substr(vdata_next.line, 1, 2) = '  ',
            "('" || replace(substr(vdata.line, 5), '|', "', '") || "'),",
            "('" || replace(substr(vdata.line, 5), '|', "', '") || "');")
        else
          iif(substr(vdata_next.line, 1, 2) = '- ',
            substr(vdata.line, 3)|| ',',
            substr(vdata.line, 3)|| ');')
      end
      from vdata left join vdata as vdata_next on vdata_next.row = vdata.row+1;

    .headers off
    .once saptabs.sql
    select * from vsql;
    .read saptabs.sql

    PRAGMA foreign_keys = ON;
    .sep ,
    CREATE TABLE if not exists ocu(
      customer_id text primary key,
      customer_unique_id,
      customer_zip_code_prefix,
      customer_city,
      customer_state
    );
    CREATE TABLE if not exists opt(
      product_category_name text primary key,
      product_category_name_english
    );
    CREATE TABLE if not exists opr(
      product_id text primary key,
      product_category_name,
      product_name_lenght,
      product_description_lenght,
      product_photos_qty,
      product_weight_g,
      product_length_cm,
      product_height_cm,
      product_width_cm
    );
    CREATE TABLE if not exists ose(
      seller_id text primary key,
      seller_zip_code_prefix,
      seller_city,
      seller_state
    );
    CREATE TABLE if not exists ogl(
      geolocation_zip_code_prefix,
      geolocation_lat,
      geolocation_lng,
      geolocation_city,
      geolocation_state
    );
    CREATE TABLE if not exists oor(
      order_id text primary key,
      customer_id references ocu,
      order_status,
      order_purchase_timestamp,
      order_approved_at,
      order_delivered_carrier_date,
      order_delivered_customer_date,
      order_estimated_delivery_date
    );
    CREATE TABLE if not exists oit(
      order_id references oor,
      order_item_id,
      product_id references opr,
      seller_id references ose,
      shipping_limit_date,
      price,
      freight_value,
      primary key(order_id, order_item_id)
    );
    CREATE TABLE if not exists opa(
      order_id references oor,
      payment_sequential,
      payment_type,
      payment_installments,
      payment_value,
      primary key(order_id, payment_sequential)
    );
    CREATE TABLE if not exists ore(
      review_id,
      order_id references oor,
      review_score,
      review_comment_title,
      review_comment_message,
      review_creation_date,
      review_answer_timestamp
    );
    CREATE VIEW VCU as select * from ocu left join ogl on
      customer_zip_code_prefix = geolocation_zip_code_prefix and
      customer_city = geolocation_city and
      customer_state = geolocation_state;
    CREATE VIEW VSE as select * from ose left join ogl on
      seller_zip_code_prefix = geolocation_zip_code_prefix and
      seller_city = geolocation_city and
      seller_state = geolocation_state;
    CREATE VIEW VPR as select * from opr left join opt on
      opr.product_category_name = opt.product_category_name;
    CREATE VIEW VORI as select * from oor left join oit on
      oor.order_id = oit.order_id;
    CREATE VIEW VORIP as select * from vori left join vpr on
      vori.product_id = vpr.product_id;
    CREATE VIEW VORS as select * from vori left join ose on
      vori.seller_id = ose.seller_id;
    CREATE VIEW VORP as select * from oor left join opa on
      oor.order_id = opa.order_id;
    CREATE VIEW VALL as select * from vorip, vorp, ocu, vors on
      vorip.order_id = vorp.order_id and vorip.customer_id = ocu.customer_id and
      vorip.order_id = vors.order_id;

    .import olist/olist_customers_dataset.csv ocu
    .import olist/olist_products_dataset.csv opr
    .import olist/product_category_name_translation.csv opt
    .import olist/olist_sellers_dataset.csv ose
    .import olist/olist_geolocation_dataset.csv ogl
    .import olist/olist_orders_dataset.csv oor
    .import olist/olist_order_items_dataset.csv oit
    .import olist/olist_order_payments_dataset.csv opa
    .import olist/olist_order_reviews_dataset.csv ore
    delete from ore where rowid = 1;
    delete from opa where rowid = 1;
    delete from oit where rowid = 1;
    delete from oor where rowid = 1;
    delete from ogl where rowid = 1;
    delete from ose where rowid = 1;
    delete from opt where rowid = 1;
    delete from opr where rowid = 1;
    delete from ocu where rowid = 1;
