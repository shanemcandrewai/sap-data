# SAP format dataset
## Fields of interest
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
    CREATE VIEW vtabs as select row, substr(line, 5) as tab from vread where line like '###%';
