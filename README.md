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
## Create SQLite initialization script from this README.md
    CREATE TABLE readme(line);
    .separator \t
    .import README.md readme
    .once initdb.sqlite
    select substr(line, 5) from readme where rowid > (select rowid from readme where line like '### Create SAP%' limit 1);
    .read initdb.sqlite
### Create SAP tables
    CREATE VIEW vread(row, line) as select rowid, line from readme;
    CREATE VIEW vsch_start as select row+1 as row from vread where line like '%## Schema%' limit 1;
    CREATE VIEW vsch_end as select row-1 as row from vread
      where row > (select * from vsch_start) and line like '## %' limit 1;
    CREATE VIEW vsch as select * from vread
      where row between (select row from vsch_start) and (select row from vsch_end);
    CREATE VIEW vsql as select
      case substr(vsch.line, 1, 3)
        when '###' then 'CREATE TABLE' || substr(vsch.line, 4) || '('
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
