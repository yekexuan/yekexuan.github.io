---
layout: post
title: "Laravel使用sqlite内存数据库进行单元测试"
date: 2016-11-03
backgrounds:
    - http://7xlrln.com1.z0.glb.clouddn.com/1147797807ace133b7o.jpg
thumb: http://7xlrln.com1.z0.glb.clouddn.com/1147797807ace133b7o.jpg?imageView2/1/w/200/h/200
categories: life
tags: home 
---

# Laravel使用sqlite内存数据库进行单元测试

## 配置测试数据库链接

在database.php配置文件中增加如下代码:

```php
'connections' => [
    // 新增一种DB链接方式, 使用sqlite内存数据库.
    'testing' => [
            'driver'   => 'sqlite',
            'database' => ':memory:',
            'prefix'   => '',
        ],
] 

```

## 修改模式数据库链接方式

修改database.php 中default设置：

```php
    // 把default链接设置为可配置.
    'default' => env('DB_CONNECTION', 'mysql'),
```

## 设置测试时使用的数据库链接

在phpunit.xml中添加DB_CONNETION的配置:
```xml
  <php>
        <env name="APP_ENV" value="testing"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="QUEUE_DRIVER" value="sync"/>
        <env name="DB_CONNECTION" value="testing"/>
    </php> 
```

## 初始化其他设置

由于使用了一些其他插件，需要进行初始化开启或关闭，都可以将参数env配置化，然后再phpunit.xml中初始化,例如关闭APP_DEBUG：

```xml 
  <php>
        <env name="APP_ENV" value="testing"/>
        <env name="APP_DEBUG" value="false"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="QUEUE_DRIVER" value="sync"/>
        <!-- 这里初始化env设置 -->
        <env name="DB_CONNECTION" value="testing"/>
    </php>
```

## 数据库初始化
由于使用内存数据库，数据无法持久化，因此运行每个测试前必须初始化数据库数据。

#### 使用Migration、factory进行数据初始化

1. 编写Migration类，声明数据库的表结构信息.
```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration {

	/**
	 * Run the migrations.
	 *
	 * @return void
	 */
	public function up()
	{
		Schema::create('wk_user', function(Blueprint $table)
		{
			$table->increments('id');
			$table->string('name');
			$table->string('email')->unique();
			$table->string('password', 60);
			$table->rememberToken();
			$table->timestamps();
		});
	}

	/**
	 * Reverse the migrations.
	 *
	 * @return void
	 */
	public function down()
	{
		Schema::drop('wk_users');
	}

}
```
2. 建立对应的Model类
```php
<?php namespace App\ADX\Model;

use Illuminate\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;

class User extends Model implements AuthenticatableContract, CanResetPasswordContract
{

    use Authenticatable, CanResetPassword;

    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'wk_user';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['email', 'password'];

    /**
     * The attributes excluded from the model's JSON form.
     *
     * @var array
     */
    protected $hidden = ['password', 'remember_token'];


}
```
3. 建立模型factory
```php
use Faker\Generator as Faker;

$factory->define(App\ADX\Model\User::class, function (Faker $faker) {
    return [
        'name'           => $faker->name,
        'email'          => $faker->safeEmail,
        'password'       => bcrypt(str_random(10)),
        'remember_token' => str_random(10),
    ];
});
```
4. 编写Test类进行单元测试:
```php 

<?php namespace Test;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use App\ADX\Model\User;

class ExampleTest extends TestCase
{

    use DatabaseMigrations;
    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        factory(User::class, 1)->create();
        /** arrange */
        $expected = 1;

        /** act */
        $actual = User::all();

        /** assert */
        $this->assertCount($expected, $actual);
        // $response = $this->call('GET', '/');

        // $this->assertEquals(404, $response->getStatusCode());
    }

}
```
5. 在phpstorm中右键test方法，或者在项目目录中执行./vendor/bin/phpunit命令，查看测试结果。
```bash
/usr/bin/php /tmp/ide-phpunit.php --configuration /home/trey/wifi/adx-be/phpunit.xml --filter "/::testBasicExample( .*)?$/" Test\ExampleTest /home/trey/wifi/adx-be/tests/ExampleTest.php
Testing started at 下午3:57 ...
PHPUnit 4.8.27 by Sebastian Bergmann and contributors.



Time: 156 ms, Memory: 16.00MB

OK (1 test, 1 assertion)

Process finished with exit code 0
```
