# DB2学習メモ


## 概要

### DB2ユーザー
1. インスタンンス所有者(db2inst1)
2. fencedユーザー(db2fenc1)
3. DB2 Administration Serverユーザー(dasusr1) 9.7で非推奨

### DB2 データ・サーバ

データベース・エンジンがインストールされたシステム。

### DB2 インスタンス

 db2inst1, db2inst2, db2inst3


## DB2 インストール

### 解凍
```
v11.1_linux64_expc.tar.gz
v11.1_linux64_nlpack.tar.gz
```
```
cd /work/
tar zxfv v11.1_linuxx64_expc.tar.gz
cd expc/
tar zxfv ../v11.1_linuxx64_nlpack.tar.gz
[root@vbfs191 expc]# ls -lh
合計 48K
drwxr-xr-x. 6 bin  bin    93 11月 10  2018 db2
-r-xr-xr-x. 1 bin  bin  5.2K 11月 10  2018 db2_deinstall
-r-xr-xr-x. 1 bin  bin  5.1K 11月 10  2018 db2_install
-r-xr-xr-x. 1 bin  bin  5.2K 11月 10  2018 db2ckupgrade
-r-xr-xr-x. 1 bin  bin  5.0K 11月 10  2018 db2ls
-r-xr-xr-x. 1 bin  bin  5.0K 11月 10  2018 db2prereqcheck
-r-xr-xr-x. 1 bin  bin  5.0K 11月 10  2018 db2setup
drwxr-xr-x. 4 root root   84  6月 11  2017 nlpack
[root@vbfs191 expc]# 
[root@vbfs191 expc]# export LANG=C
[root@vbfs191 expc]# ./db2setup -f sysreq
```




## インスタンスの作成

### OSユーザ作成
```bash
useradd -g db2iadm1 db2inst2
echo db2inst2 | passwd db2inst2 --stdin
useradd -g db2iadm1 db2inst3
echo db2inst3 | passwd db2inst3 --stdin
```

### インスタンス作成
```bash
/opt/ibm/db2/V11.1/instance/db2icrt -u db2fenc1 db2inst2
/opt/ibm/db2/V11.1/instance/db2icrt -u db2fenc1 db2inst3
````

### データベースマネージャのコンフィグ
```bash
db2 get dbm cfg
```

### サンプルデータベース作成
```bash
db2sampl
```

### サンプルデータベースのコンフィグ
```bash
db2 get db cfg for sample
```

### DB2プロファイルの表示
```bash
db2set -all
```

### 接続
```bash
db2 connect to sample
```

## ホスト名変更

```bash
export DB2INSTANCE=db2inst1
/home/db2inst1/sqllib/adm/db2set -g DB2SYSTEM=centst9x1
export DB2INSTANCE=db2inst2
/home/db2inst2/sqllib/adm/db2set -g DB2SYSTEM=centst9x1
export DB2INSTANCE=db2inst3
/home/db2inst3/sqllib/adm/db2set -g DB2SYSTEM=centst9x1
```

## db2topを使用する方法

```bash
ln -s /usr/lib64/libncurses.so.6.2 /usr/lib64/libncurses.so.5
```



## 権限

### ロールの一覧
```sql
db2 "select * from syscat.roles"
```

### ロールのメンバー一覧
```sql
db2 "select * from syscat.roleauth"
```
### 各テーブルのGRANT(特権)一覧
```sql
db2 "select * from syscat.tabauth"
```
### 各スキーマのGRANT一覧
```sql
db2 "select * from syscat.schemaauth"
```
### 権限一覧
GRANTEEのPUBLICは、全ユーザが対象なので基本的な権限を確認しておいたほうがよい。

```sql
db2 "select * from syscat.dbauth"
db2 "select * from sysibm.sysdbauth"
```

### ユーザ作成
```sql
useradd db2user1
echo db2user1 | passwd db2user1 --stdin
useradd db2user2
echo db2user2 | passwd db2user2 --stdin
useradd db2user3
echo db2user3 | passwd db2user3 --stdin



db2 "GRANT DBADM,CREATETAB,BINDADD,CONNECT,CREATE_NOT_FENCED,IMPLICIT_SCHEMA,LOAD ON DATABASE TO USER DB2USER1"
db2 "GRANT DBADM,CREATETAB,BINDADD,CONNECT,CREATE_NOT_FENCED,IMPLICIT_SCHEMA,LOAD ON DATABASE TO USER DB2USER2"
```


## db2 監査

・監査ポリシー
・アクティブ監査ログ
・アーカイブ監査ログ
・レポーティング

### 監査ポリシー作成
### 監査ログ・アーカイブ監査ログ出力先設定
```sql
db2audit configure datapath /db/dbaud/audit archivepath /db/dbaud/auditarch
```

### 確認
```sql
db2audit describe
```

### ポリシー作成
```sql
db2 "create audit policy policy1 categories execute status both error type audit"
db2 "create audit policy policy1 categories execute status both error type normal"
```

### ポリシーのDBへのアタッチ
```sql
db2 "audit database using policy policy1"
```

### 監査ポリシーの確認
```sql
db2 "select varchar(auditpolicyname,20) as auditpolicyname,auditstatus,contextstatus,validatestatus,checkingstatus,secmaintstatus,objmaintstatus,sysadminstatus,executestatus,executewithdata,errortype from syscat.auditpolicies"

db2 "select varchar(auditpolicyname,16) as auditpolicyname,objectname,objectschema,objecttype from syscat.audituse"

db2 "select * from syscat.auditpolicies"

db2 "select * from syscat.audituse"

db2look -ap -d sample
```

### 監査ポリシーの削除
```sql
db2 "audit database remove policy"
db2 "drop audit policy policy1"

db2 "select * from syscat.audituse"

db2 "select varchar(auditpolicyname,20) as auditpolicyname,auditpolicyid,varchar(objecttype,20) as objecttype,varchar(subobjecttype,20) as subobjecttype,varchar(objectschema,20) as objectschema,varchar(objectname,20) as objectname,varchar(auditexceptionenabled,20) as auditexceptionenabled from syscat.audituse"

AUDITPOLICYNAME      AUDITPOLICYID OBJECTTYPE           SUBOBJECTTYPE        OBJECTSCHEMA         OBJECTNAME           AUDITEXCEPTIONENABLED
-------------------- ------------- -------------------- -------------------- -------------------- -------------------- ---------------------
ACCESSTOSECRETPOLICY           100                                           -                    CURRENT SERVER       N
```

### 監査ログファイルの確認
```sql
[db2inst1@vbfs192 ~]$ ls -lh sqllib/security/auditdata/
合計 12K
-rw-------. 1 db2inst1 db2iadm1 11K 11月 26 02:05 db2audit.db.SAMPLE.log.0
[db2inst1@vbfs192 ~]$
```

### アーカイブ監査ログ
### ログのフラッシュ
```bash
db2audit flush
```

### 監査ログのアーカイブ
```bash
db2audit archive database sample to /home/db2inst1/audit/
db2audit archive database sample to /home/db2inst2/audit/
db2audit archive database sample to /home/db2inst3/audit/
```

##レポーティング
### ディレクトリ指定
```bash
db2audit extract file report_$(date +%Y%m%d_%H%M%S).txt from path /home/db2inst2/audit files "*"
```

### ファイル指定
```bash
db2audit extract file db2audit.txt from files “*”
db2audit extract file report.txt from files db2audit.db.SAMPLE.log.0.20231121234845
```

### CSV出力
```bash
db2audit extract deals to /db/dbaudit/work from files “*”  <- NG
```



## CSVをインポート
別途テーブルの定義は作成する。

### インポートコマンド
```sql
db2 "import from 00_zenkoku_all_20230929.csv of del commitcount 100000 insert into ZENKOKU"
db2 "import from 40_fukuoka_all_20230929.csv of del commitcount 100000 insert into FUKUOKA"
```

### IXFをインポート

```sql
db2 "IMPORT FROM zenkoku.ixf OF IXF MODIFIED BY FORCECREATE MESSAGES import_zenkoku.msg CREATE INTO ZENKOKU"

```

## LOAD

```sql
db2look -d sample -e -t zenkoku -o create_zenkoku.ddl
db2 -tvf create_zenkoku.ddl
db2 "select count(*) from zenkoku"

```



```sql
db2 "load from zenkoku.ixf of ixf replace into ZENKOKU"
```





## テーブル作成

```
db2 "create table tokyo as (select * from zenkoku where prefecturename = '東京都') with data"
```

## Exportファイルをインポート

### テーブル定義を含めてインポート
```sql
date; db2 "import from zenkoku.ixf of ixf commitcount 100000 messages zenkoku_imp.txt create into zenkoku"; date
date; db2 "import from zenkoku.ixf of ixf commitcount 100000 messages zenkoku_imp2.txt create into zenkoku2"; date
```

## テーブルを別のテーブルスペースに移動

```sql
db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','ZENKOKU','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','MOVE,TRACE')"

db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','ZENKOKU','IBMDB2SAMPLEREL','IDXSPACE1','IBMDB2SAMPLEREL','','','','','','MOVE,TRACE')"
db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO','IBMDB2SAMPLEREL','IDXSPACE1','IBMDB2SAMPLEREL','','','','','','MOVE,TRACE')"


db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','MOVE,TRACE')"


db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO2','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','INIT,TRACE')"
db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO2','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','COPY,TRACE')"
db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO2','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','REPLAY,TRACE')"
db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO2','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','SWAP,TRACE')"

# swapが失敗した場合
db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO2','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','CLEANUP,TRACE')"

db2 "CALL SYSPROC.ADMIN_MOVE_TABLE('DB2INST1','TOKYO2','NEWSPACE1','IDXSPACE1','NEWSPACE1','','','','','','CANCEL,TRACE')"

db2 "select TBSPACEID from syscat.tables where TABNAME='ZENKOKU'"

db2 "select TBSPACE from syscat.tablespaces where TBSPACEID = 3"

db2 "select TBSPACE from syscat.tablespaces where TBSPACEID = (select TBSPACEID from syscat.tables where TABNAME='TOKYO')"

db2 "select TBSPACE from syscat.tablespaces where TBSPACEID = (select TBSPACEID from syscat.tables where TABNAME='ZENKOKU')"
```

## テーブルスペースの管理

### テーブルスペースの追加
```sql
db2 "create tablespace newspace1"
db2 "create tablespace idxspace1"
```

### インデックスの作成
```sql
db2 "create index zenkoku_name_idx on zenkoku (name)"
db2 "create index aichi_name_idx on aichi (name)"
db2 "create index fukuoka_name_idx on fukuoka (name)"
db2 "create index hyougo_name_idx on hyougo (name)"
db2 "create index kagoshima_name_idx on kagoshima (name)"
db2 "create index miyazaki_name_idx on miyazaki (name)"
db2 "create index nigata_name_idx on nigata (name)"
db2 "create index osaka_name_idx on osaka (name)"
```

### インデックスの確認
```sql
db2 "SELECT TABNAME, INDNAME, COLNAMES FROM SYSCAT.INDEXES WHERE TABNAME = 'TEST'"
db2 "select tabname, indname, colnames from syscat.indexes"
db2 "SELECT TABNAME, INDNAME, COLNAMES FROM SYSCAT.INDEXES WHERE INDNAME LIKE '%NAME_IDX'"
```

### テーブルスペースの確認1
```sql
db2 list tablespaces show detail
```

### テーブルスペースの確認2
```bash
db2pd -db sample -tablespace
```

### ファイルサイズの確認
```bash
ls -lh /home/db2inst1/db2inst1/NODE0000/SAMPLE/T000000?/*
```

### テーブルスペースを最大限縮小
```sql
db2 "alter tablespace ibmdb2samplerel reduce max"
```

### テーブルスペースを25%縮小
```sql
db2 "alter tablespace ibmdb2samplerel reduce 25 percent"
```

## Reorg & Runstats

### テーブルの統計情報確認
```sql
db2 "reorgchk current statistics on table zenkoku"
db2 "reorgchk current statistics on table aichi"
```

### テーブルのReorg実施
```sql
db2 "reorg table zenkoku"
db2 "reorg table aichi"
```

### テーブルのすべてのインデックスを再構成
```sql
db2 "reorg indexes all for table db2inst1.zenkoku"
db2 "reorg indexes all for table db2inst1.aichi"
```

## バックアップ
```bash
mkdir /backup
```

### オフラインバックアップ
```sql
db2 "backup database sample to /home/db2inst1/backup/"
db2 "backup database sample to /backup"
```

## データ操作

### テーブル定義確認
```sql
db2 describe table zenkoku
```

### Query
```sql
db2 "select count(*) from aichi"
db2 "select count(*) from aichi where name like '有限会社%'"
db2 "select count(*) from aichi where name like '%有限会社'"
db2 "select count(*) from zenkoku"
db2 "select count(*) from zenkoku where PREFECTURENAME = '東京都'"
db2 "select count(*) from zenkoku where PREFECTURENAME = '神奈川県'"
db2 "select count(*) from zenkoku where PREFECTURENAME = '愛知県'"
db2 "select count(*) from zenkoku where PREFECTURENAME = '大阪府'"
db2 "select count(*) from zenkoku where PREFECTURENAME = '京都府'"
db2 "select count(*) from zenkoku where PREFECTURENAME = '兵庫県'"
db2 "select count(*) from zenkoku where PREFECTURENAME = '福岡県'"
```

### 行削除
```sql
db2 "delete from aichi where name like '有限会社%'"
db2 "delete from aichi where name like '%有限会社'"
db2 "delete from zenkoku where PREFECTURENAME = '東京都'"
db2 "delete from zenkoku where PREFECTURENAME = '神奈川県'"
date; db2 "delete from zenkoku where PREFECTURENAME = '愛知県'"; date
date; db2 "delete from zenkoku where PREFECTURENAME = '大阪府'"; date
date; db2 "delete from zenkoku where PREFECTURENAME = '京都府'"; date
date; db2 "delete from zenkoku where PREFECTURENAME = '京都府'"; date
date; db2 "delete from zenkoku where PREFECTURENAME = '兵庫県'"; date
date; db2 "delete from zenkoku where PREFECTURENAME = '福岡県'"; date
```

### ログのアーカイブ
```sql
db2 archive log for database sample
```

## リストア
```bash
db2set -all
db2set DB2_RESTORE_GRANT_ADMIN_AUTHORITIES=ON
db2set -all

db2 "RESTORE DATABASE SAMPLE FROM /backup TAKEN AT 20240115213512 TO /home/db2inst2 WITHOUT ROLLING FORWARD"
db2 "RESTORE DATABASE SAMPLE FROM /backup TAKEN AT 20240115213512 ON /home/db2inst2 WITHOUT ROLLING FORWARD"
db2 "RESTORE DATABASE SAMPLE FROM /backup TAKEN AT 20240116014003 ON /home/db2inst3 DBPATH ON /home/db2inst3 WITHOUT ROLLING FORWARD"
```



## チューニングメモ

書き出しアルゴリズムの設定

```bash
db2set DB2_USE_ALTERNATE_PAGE_CLEANING=ON
```

