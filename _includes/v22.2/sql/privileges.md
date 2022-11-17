Privilege | Levels
----------|------------
`ALL` | Database, Schema, Table, Type
`CREATE` | Database, Schema, Table
`DROP` | Database, Table
`EXECUTE` | Function
`GRANT` | Database, Function, Schema, Table, Type
`CONNECT` | Database
`SELECT` | Table
`INSERT` | Table
`DELETE` | Table
`UPDATE` | Table
`USAGE`  | Function, Schema, Type
`ZONECONFIG` | Database, Table
`EXTERNALCONNECTION` | System
`BACKUP` | System, Database, Table
`RESTORE` | System, Database
`EXTERNALIOIMPLICITACCESS` | System
`MODIFYCLUSTERSETTING` | <a name="modifyclustersetting"></a> [Global privilege](../grant.html#grant-global-privileges-on-the-entire-cluster) that allows users to use the [`SET CLUSTER SETTING`](../set-cluster-setting.html) statement.
