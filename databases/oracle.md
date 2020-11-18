# Oracle Network Configuration (listener.ora , tnsnames.ora , sqlnet.ora)
## listener.ora
The "listerner.ora" file contains **server** side network configuration parameters. It can be found in the "$ORACLE_HOME/network/admin" directory on the server. Here is an example of a basic "listener.ora" file from Linux. We can see the listener has the default name of "LISTENER" and is listening for TCP connections on port 1521. Notice the reference to the hostname "myserver.example.com". If this is incorrect, the listener will not function correctly.
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1)) 
      (ADDRESS = (PROTOCOL = TCP)(HOST = myserver.example.com)(PORT = 1521))
    )
  )
```
After the "listener.ora" file is amended the listener should be restarted or reloaded to allow the new configuration to take effect.
```
$ # Restart
$ lsnrctl stop
$ lsnrctl start

$ # Or Reload.
$ lsnrctl reload
```
The listener defined above doesn't have any services defined. These are created when database instances auto-register with it. In some cases you may want to manually configure services, so they are still visible even when the database instance is down. If this is the case, you may use a "listener.ora" file like the following.
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1)) 
      (ADDRESS = (PROTOCOL = TCP)(HOST = myserver.example.com)(PORT = 1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl.example.com)
      (ORACLE_HOME = /u01/app/oracle/product/11.2.0.4/db_1)
      (SID_NAME = orcl)
    )
  )
```
If there are multiple database instances on the server, you can added multiple SID_DESC entries inside the SID_LIST section.
## tnsnames.ora
The "tnsnames.ora" file contains **client** side network configuration parameters. It can be found in the "$ORACLE_HOME/network/admin" directory on the client. This file will also be present on the server if client style connections are used on the server itself. Here is an example of a "tnsnames.ora" file.
```
LISTENER = (ADDRESS = (PROTOCOL = TCP)(HOST = myserver.example.com)(PORT = 1521))

orcl.example.com =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = myserver.example.com)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = orcl)
    )
  )
```
The alias used at the start of the entry can be whatever you want. It doesn't have to match the name of the instance or service. Notice the PROTOCOL, HOST and PORT match that of the listener. The SERVICE_NAME can be any valid service presented by the listener. You can check the available services by issuing the lsnrctl status or lsnrctl service commands on the database server. Typically there is at least one service matching the ORACLE_SID of the instance, but you can create more.
## sqlnet.ora
The "sqlnet.ora" file contains client side network configuration parameters. It can be found in the "$ORACLE_HOME/network/admin" directory on the client. This file will also be present on the server if client style connections are used on the server itself, or if some additional server connection configuration is required. Here is an example of an "sqlnet.ora" file.
```
NAMES.DIRECTORY_PATH= (TNSNAMES, ONAMES, HOSTNAME)
NAMES.DEFAULT_DOMAIN = example.com

# The following entry is necessary on Windows if OS authentication is required.
SQLNET.AUTHENTICATION_SERVICES= (NTS)
```
There are lots of parameters that can be added to control tracing, encryption, wallet locations etc. These are out of the scope of this article.

Resource: [https://oracle-base.com/articles/misc/oracle-network-configuration]