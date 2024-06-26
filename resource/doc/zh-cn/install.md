# 环境需求

* PHP >= 7.2
* [Composer](https://getcomposer.org/) >= 2.0


### 1. 创建项目

```php
composer create-project workerman/webman
```

### 2. 运行

进入webman目录   

#### windows用户
双击 `windows.bat` 或者运行 `php windows.php` 启动

> **提示**
> 如果有报错，很可能是有函数被禁用，参考[函数禁用检查](others/disable-function-check.md)解除禁用

#### linux用户
调试方式运行（用于开发调试，打印数据会显示在终端，终端关闭后webman服务也随之关闭）
 
```php
php start.php start
```

守护进程方式运行（用于正式环境，打印数据不会显示在终端，终端关闭后webman服务会持续运行）

```php
php start.php start -d
```

> **提示**
> 如果有报错，很可能是有函数被禁用，参考[函数禁用检查](others/disable-function-check.md)解除禁用

### 3.访问

浏览器访问 `http://ip地址:8787`


