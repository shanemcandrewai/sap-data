# SAP format dataset
## Schema
### T002
- SPRAS primary key
### KNA1
- KUNNR primary key
- LAND1
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
### KNA1
    0000500000|IT|Guccion Gucci S.p.A|gucci50123|20100201|sandrag
    0000500001|IT|Prada S.p.A|prada20135|20110303|accardia
    0000500002|UK|River Island Clothing co. Limited|rive636095|20100305|smithds
    0000500003|UK|Boohoo Group plc|boohoo7297|20100115|sandrag
### VBAK
    0015000000|20100215|15:15:01|sandrag|20100215|OR|SDNL|000050001
    0015000001|20100216|12:14:44|sandrag|20100216|OR|SDNL|000050002
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
## Create and execute SQLite initialization script from this README.md
    DROP TABLE if exists readme;
    CREATE TABLE readme(line);
    .separator \t
    .import README.md readme
    .once initdb.sqlite
    select substr(line, 5) from readme where rowid > (select rowid from readme where line like '### Create and%' limit 1);
    .read initdb.sqlite
### Create and populate SAP tables
    CREATE VIEW if not exists VREAD(row, line) as select rowid, line from readme;
    CREATE VIEW if not exists VDATA as select * from vread
      where row between
        (select row+1 from vread where line like '%## Schema%' limit 1) and
        (select row-1 from vread where line like '%## Create%' limit 1);
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
