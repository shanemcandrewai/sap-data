# SAP format dataset
## Schema
### KNA1
- KUNNR primary key
- LAND1
- NAME1
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
### VBAP
- VBELN references VBAK
- POSNR
- MATNR references MARA
- PMATN
- MEINS
- primary key(VBELN, POSNR)
## sqlite3
    PRAGMA foreign_keys = ON;
    CREATE TABLE readme(line);
    .import README.md readme
    CREATE VIEW vread(row, line) as select rowid, line from readme;
    CREATE VIEW vsch_start as select row+1 as row from vread where line like '%## Schema%' limit 1;
    CREATE VIEW vsch_end as select row-1 as row from vread where row > (select * from vsch_start) and line like '## %' limit 1;
    CREATE VIEW vsch as select * from vread where row between (select row from vsch_start) and (select row from vsch_end);
    CREATE VIEW vtabs as select row, substr(line, 5) as tab from vsch where line like '##%'
    CREATE VIEW vfields as select row, substr(line, 3) as field from vsch where line like '- %';
