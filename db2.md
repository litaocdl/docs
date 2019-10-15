1. Db2 can execute query using command line but can not through jcc and report error 
```
DB2 SQL Error: SQLCODE=-20249, SQLSTATE=     , SQLERRMC=NULLID.SYSSH100, DRIVER=4.26.14,
```
solution

```
db2 connect to mydb
db2 bind $(eval ~${DB2INSTANCE})/sqllib/bnd/@db2cli.lst  blocking all grant public

```

```
db2 ? sqln20249


SQL20249N  The statement was not processed because the package named
      "<package-name>" needs to be explicitly rebound.

db2 rebind NULLID.SYSSH100
```
