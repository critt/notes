##### Dump sqlite db table to file
```
critt@Mac injured % sqlite3 Thisisatest.db
SQLite version 3.43.2 2023-10-10 13:08:14
Enter ".help" for usage hints.
sqlite> .open Thisisatest.db
sqlite> .mode csv
sqlite> .headers on
sqlite> .out file.csv
sqlite> select * from Thisisatest;
sqlite>
```

