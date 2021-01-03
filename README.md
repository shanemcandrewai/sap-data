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
- BSTME
### MAKT
- MATNR references MARA
- SPRAS references T002
- MAKTX
### VBAP
- VBELN references VBAK
- POSNR
- MATNR references MARA
- PMATN
- MEINS
- primary key(VBELN, POSNR)
## Table contents
### T002
EN
### KNA1
0000500000|IT|Guccion Gucci S.p.A|gucci50123|20100201|sandrag
0000500001|IT|Prada S.p.A|prada20135|IT|20110303|accardia
0000500002|UK|River Island Clothing co. Limited|rive636095|20100305|smithds
0000500003|UK|Boohoo Group plc|boohoo7297|20100115|sandrag
### VBAK
0015000000|20100215|15:15:01|sandrag|20100215|OR|SDNL|000050001
0015000001|20100216|12:14:44|sandrag|20100216|OR|SDNL|000050002
### MARA
000000000400000000|20100901|gallage|20180106|simonr||BAG1
## Create SQLite initialization script from this README.md
    CREATE TABLE if not exists readme(line);
    .separator \t
    .import README.md readme
    .once initdb.sqlite
    select substr(line, 5) from readme where rowid > (select rowid from readme where line like '### Create SAP%' limit 1);
    .read initdb.sqlite
### Create SAP tables
    CREATE VIEW if not exists vread(row, line) as select rowid, line from readme;
    CREATE VIEW if not exists vsch_start as select row+1 as row from vread
      where line like '%## Schema%' limit 1;
    CREATE VIEW if not exists vsch_end as select row-1 as row from vread
      where row > (select * from vsch_start) and line like '## %' limit 1;
    CREATE VIEW if not exists vsch as select * from vread
      where row between (select row from vsch_start) and
                        (select row from vsch_end);
    CREATE VIEW if not exists vsql as select
      case substr(vsch.line, 1, 3)
        when '###' then 'CREATE TABLE if not exists' || substr(vsch.line, 4) || '('
        else
          case substr(vsch_next.line, 1, 1)
            when '-' then substr(vsch.line, 3)|| ','
            else substr(vsch.line, 3)|| ');'
          end
      end
      as sql from vsch left join vsch as vsch_next on vsch_next.row = vsch.row+1;
    .headers off
    .once saptabs.sql
    select * from vsql;
    .read saptabs.sql
