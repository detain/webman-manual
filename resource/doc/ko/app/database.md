# 데이터베이스

대부분의 플러그인이 [webman-admin](https://www.workerman.net/plugin/82)을 설치하므로 `webman-admin`의 데이터베이스 설정을 직접 재사용하는 것을 권장합니다.

모델 기본 클래스로 `plugin\admin\app\model\Base`를 사용하면 webman-admin의 데이터베이스 설정을 자동으로 사용합니다.
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

`plugin.admin.mysql`을 통해 webman-admin 데이터베이스에 접근할 수도 있습니다. 예:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## 자체 데이터베이스 사용

플러그인은 자체 데이터베이스를 사용하도록 선택할 수 있습니다. 예를 들어 `plugin/foo/config/database.php` 내용:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql은 연결 이름
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => '데이터베이스',
            'username'    => '사용자명',
            'password'    => '비밀번호',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin은 연결 이름
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => '데이터베이스',
            'username'    => '사용자명',
            'password'    => '비밀번호',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

참조 형식은 `Db::connection('plugin.{플러그인}.{연결명}');`입니다. 예:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

메인 프로젝트의 데이터베이스를 사용하려면 직접 호출하세요:

```php
use support\Db;
Db::table('user')->first();
// 메인 프로젝트에 admin 연결도 설정되어 있다고 가정
Db::connection('admin')->table('admin')->first();
```

#### Model용 데이터베이스 설정

Model용 Base 클래스를 만들고 `$connection` 속성으로 플러그인 자체의 데이터베이스 연결을 지정할 수 있습니다:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

이렇게 하면 플러그인의 모든 Model이 Base를 상속하여 플러그인 자체의 데이터베이스를 자동으로 사용합니다.

## 데이터베이스 자동 가져오기
`php webman app-plugin:create foo`를 실행하면 foo 플러그인이 자동 생성되며 `plugin/foo/api/Install.php`와 `plugin/foo/install.sql`이 포함됩니다.

> **팁**
> install.sql 파일이 생성되지 않으면 `webman/console` 업그레이드를 시도하세요: `composer require webman/console ^1.3.6`

#### 플러그인 설치 시 데이터베이스 가져오기
플러그인 설치 시 Install.php의 `install` 메서드가 실행되며, 이 메서드는 `install.sql`의 SQL 문을 자동으로 실행하여 데이터베이스 테이블을 가져옵니다.

`install.sql` 내용은 테이블 생성 및 테이블의 과거 변경 SQL 문이며, 각 문은 `;`로 끝나야 합니다. 예:
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '기본키',
  `order_id` varchar(50) NOT NULL COMMENT '주문ID',
  `user_id` int NOT NULL COMMENT '사용자ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT '결제금액',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='주문';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '기본키',
  `name` varchar(50) NOT NULL COMMENT '이름',
  `price` int NOT NULL COMMENT '가격',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='상품';
```

**데이터베이스 연결 변경**

`install.sql`의 가져오기 대상 데이터베이스는 기본적으로 webman-admin의 데이터베이스입니다. 다른 데이터베이스로 가져오려면 `Install.php`의 `$connection` 속성을 수정하세요. 예:
```php
<?php

class Install
{
    // 플러그인 자체의 데이터베이스 지정
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**테스트**

`php webman app-plugin:install foo`를 실행하여 플러그인을 설치한 후 데이터베이스를 확인하면 `foo_orders`와 `foo_goods` 테이블이 생성되어 있을 것입니다.

#### 플러그인 업그레이드 시 테이블 구조 변경
플러그인 업그레이드로 새 테이블 생성 또는 테이블 구조 변경이 필요한 경우 `install.sql` 끝에 해당 문을 추가하세요. 각 문은 `;`로 끝나야 합니다. 예: `foo_user` 테이블 추가 및 `foo_orders` 테이블에 `status` 컬럼 추가
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '기본키',
  `order_id` varchar(50) NOT NULL COMMENT '주문ID',
  `user_id` int NOT NULL COMMENT '사용자ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT '결제금액',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='주문';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT '기본키',
 `name` varchar(50) NOT NULL COMMENT '이름',
 `price` int NOT NULL COMMENT '가격',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='상품';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT '기본키',
 `name` varchar(50) NOT NULL COMMENT '이름'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='사용자';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT '상태';
```

업그레이드 시 Install.php의 `update` 메서드가 실행되며, 마찬가지로 `install.sql`의 문을 실행합니다. 새 문이 있으면 실행하고, 이미 적용된 문은 건너뛰어 업그레이드 시 데이터베이스 변경이 올바르게 적용됩니다.

#### 플러그인 제거 시 데이터베이스 삭제
플러그인 제거 시 Install.php의 `uninstall` 메서드가 호출됩니다. 이 메서드는 `install.sql`의 CREATE TABLE 문을 자동 분석하고 해당 테이블을 삭제하여 플러그인 제거 시 데이터베이스 테이블을 삭제합니다.
제거 시 자체 `uninstall.sql`만 실행하고 자동 테이블 삭제를 원하지 않으면 `plugin/플러그인명/uninstall.sql`을 생성하세요. 그러면 `uninstall` 메서드는 `uninstall.sql` 파일의 문만 실행합니다.
