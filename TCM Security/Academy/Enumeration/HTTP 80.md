
![](attachments/Pasted%20image%2020251125152219.png)

I did a `dirbuster` and I found these directories:
```
DirBuster 1.0-RC1 - Report
http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project
Report produced on Tue Nov 25 08:37:45 EST 2025
--------------------------------

http://192.168.126.132:80
--------------------------------
Directories found during testing:

Dirs found with a 200 response:

/
/academy/
/academy/assets/
/academy/admin/
/academy/assets/js/
/academy/assets/img/
/academy/admin/assets/
/academy/admin/assets/js/
/academy/assets/css/
/academy/assets/fonts/
/academy/admin/assets/img/
/academy/includes/
/academy/admin/assets/css/
/academy/admin/assets/fonts/
/academy/db/
/academy/admin/includes/
/phpmyadmin/

Dirs found with a 403 response:

/icons/
/icons/small/
/phpmyadmin/templates/
/phpmyadmin/themes/
/phpmyadmin/doc/
/phpmyadmin/doc/html/
/phpmyadmin/examples/
/phpmyadmin/js/
/phpmyadmin/libraries/
/phpmyadmin/vendor/
/phpmyadmin/doc/html/_images/
/phpmyadmin/vendor/google/
/phpmyadmin/js/vendor/
/phpmyadmin/setup/lib/
/phpmyadmin/sql/
/phpmyadmin/themes/original/
/phpmyadmin/themes/original/img/
/phpmyadmin/themes/original/css/
/phpmyadmin/locale/
/phpmyadmin/locale/de/
/phpmyadmin/locale/fr/
/phpmyadmin/locale/it/
/phpmyadmin/locale/nl/
/phpmyadmin/locale/uk/
/phpmyadmin/locale/es/
/phpmyadmin/locale/cs/
/phpmyadmin/locale/pl/
/phpmyadmin/locale/id/
/phpmyadmin/locale/tr/
/phpmyadmin/locale/ca/
/phpmyadmin/locale/ru/
/phpmyadmin/locale/pt/
/phpmyadmin/locale/ja/
/phpmyadmin/locale/bg/
/phpmyadmin/locale/ar/
/phpmyadmin/locale/hu/
/phpmyadmin/locale/vi/
/phpmyadmin/locale/fi/
/phpmyadmin/locale/gl/
/phpmyadmin/locale/be/
/phpmyadmin/js/designer/
/phpmyadmin/locale/si/
/phpmyadmin/locale/da/
/phpmyadmin/locale/sv/
/phpmyadmin/locale/th/
/phpmyadmin/locale/el/
/phpmyadmin/locale/ko/
/phpmyadmin/locale/sk/
/phpmyadmin/locale/sl/
/phpmyadmin/locale/ia/
/phpmyadmin/locale/ro/
/phpmyadmin/locale/lt/
/phpmyadmin/locale/he/
/phpmyadmin/locale/az/
/phpmyadmin/locale/et/
/phpmyadmin/locale/bn/
/phpmyadmin/locale/nb/

Dirs found with a 401 response:

/phpmyadmin/setup/


--------------------------------
Files found during testing:

Files found with a 200 responce:

/academy/index.php
/academy/assets/js/jquery-1.11.1.js
/academy/admin/index.php
/academy/assets/js/bootstrap.js
/academy/admin/assets/js/bootstrap.js
/academy/admin/assets/js/jquery-1.11.1.js
/academy/assets/css/bootstrap.css
/academy/assets/css/font-awesome.css
/academy/assets/css/style.css
/academy/assets/fonts/FontAwesome.otf
/academy/includes/config.php
/academy/assets/fonts/fontawesome-webfont.eot
/academy/includes/footer.php
/academy/assets/fonts/fontawesome-webfont.svg
/academy/assets/fonts/fontawesome-webfont.ttf
/academy/includes/header.php
/academy/assets/fonts/fontawesome-webfont.woff
/academy/includes/menubar.php
/academy/admin/assets/css/font-awesome.css
/academy/admin/assets/css/bootstrap.css
/academy/assets/fonts/fontawesome-webfont.woff2
/academy/admin/assets/css/style.css
/academy/assets/fonts/glyphicons-halflings-regular.eot
/academy/admin/assets/fonts/FontAwesome.otf
/academy/assets/fonts/glyphicons-halflings-regular.svg
/academy/assets/fonts/glyphicons-halflings-regular.ttf
/academy/admin/assets/fonts/fontawesome-webfont.eot
/academy/assets/fonts/glyphicons-halflings-regular.woff
/academy/admin/assets/fonts/fontawesome-webfont.svg
/academy/assets/fonts/glyphicons-halflings-regular.woff2
/academy/admin/assets/fonts/fontawesome-webfont.woff
/academy/db/onlinecourse.sql
/academy/admin/assets/fonts/fontawesome-webfont.ttf
/academy/admin/assets/fonts/fontawesome-webfont.woff2
/academy/admin/assets/fonts/glyphicons-halflings-regular.eot
/academy/admin/assets/fonts/glyphicons-halflings-regular.svg
/academy/admin/assets/fonts/glyphicons-halflings-regular.ttf
/academy/admin/assets/fonts/glyphicons-halflings-regular.woff
/academy/admin/assets/fonts/glyphicons-halflings-regular.woff2
/academy/admin/includes/header.php
/academy/admin/includes/menubar.php
/academy/admin/includes/footer.php
/academy/admin/includes/config.php
/academy/logout.php
/academy/admin/logout.php
/phpmyadmin/index.php
/phpmyadmin/themes.php
/phpmyadmin/ajax.php
/phpmyadmin/navigation.php
/phpmyadmin/license.php
/phpmyadmin/logout.php
/phpmyadmin/changelog.php
/phpmyadmin/export.php
/phpmyadmin/robots.txt
/phpmyadmin/js/messages.php
/phpmyadmin/sql.php
/phpmyadmin/import.php
/phpmyadmin/examples/signon.php

Files found with a 302 responce:

/academy/print.php
/academy/admin/print.php
/academy/admin/course.php
/academy/admin/department.php
/phpmyadmin/url.php
/academy/enroll.php
/academy/admin/session.php


--------------------------------
```

So I went to `http://192.168.126.132/academy/index.php`  

![](attachments/Pasted%20image%2020251125153914.png)

and I put the credentials that I found from FTP. So I put `10201321` and `student` and I got in.

![](attachments/Pasted%20image%2020251125154016.png)

