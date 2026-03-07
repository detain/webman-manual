# التحقق من الدوال المعطلة

استخدم هذا السكريبت للتحقق من وجود دوال معطلة. نفّذ الأمر التالي في سطر الأوامر: ```curl -Ss https://www.workerman.net/webman/check | php```

إذا ظهرت الرسالة ```Functions اسم_الدالة has been disabled. Please check disable_functions in php.ini``` فهذا يعني أن الدوال التي يعتمد عليها webman معطلة، ولا بد من إلغاء تعطيلها في ملف php.ini لاستخدام webman بشكل صحيح.
لإلغاء التعطيل، اختر إحدى الطرق التالية.

## الطريقة الأولى
تثبيت `webman/console`
```
composer require webman/console ^v1.2.35
```

نفّذ الأمر التالي
```
php webman fix-disable-functions
```

## الطريقة الثانية

نفّذ السكريبت `curl -Ss https://www.workerman.net/webman/fix-disable-functions | php` لإلغاء التعطيل

## الطريقة الثالثة

نفّذ الأمر `php --ini` للعثور على موقع ملف php.ini المستخدم من طرف php cli

افتح ملف php.ini، وابحث عن `disable_functions`، وأزل الدوال التالية من قائمة التعطيل:
```
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
pcntl_wait
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
proc_open
proc_get_status
proc_close
shell_exec
exec
putenv
getenv
```
