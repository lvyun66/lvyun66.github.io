---
title: Composer自动加载原理
tags:
    - PHP
    - Composer
categories:
    - Composer
---

# composer自动加载原理

### 1. composer 介绍

composer 是 PHP 管理依赖的工具,而不是包管理工具;

你可以在自己的项目中声明所依赖的外部工具库(libraries),Composer 会自动帮你安装这些依赖库;

### 2. composer自动加载方式

composer 自动加载有四种方式:

	1. classMap
	2. psr-0
	3. psr-4
	4. files

目前 composer 这四种加载方式都会用到,但最流行的,最常用的是 psr-4

1. classMap

	```
    "autoload": {
        "classmap": [
            "src/"
        ]
    },
	```
	composer会自动读取这个文件下面所有的文件,然后在 vendor/composer/autoload_classmap.php中将所有的 class的namespace+classname生成一个key=>value 数组.

	```
	    'PHPUnit\\Framework\\TestCase' => $vendorDir . '/phpunit/phpunit/src/ForwardCompatibility/TestCase.php',
	```

2. psr-0

	```
	现在这个标准已经废弃

	"autoload": {
        "psr-0": {"JsonMapper": "src/"}
    },
	```
	对应文件夹如下:

	```
	└── jsonmapper
    ├── ChangeLog
    ├── README.rst
    ├── composer.json
    ├── example
    │   ├── Address.php
    │   ├── Contact.php
    │   ├── run.php
    │   └── single.json
    ├── package.xml
    └── src
        ├── JsonMapper
        │   └── Exception.php
        └── JsonMapper.php

	```

	按照 psr-0 的规则, 当自动加载JsonMapper这个 class 时,会去寻找vendor/jsonmapper/src/JsonMapper.php, 最终这个配置会以 Map 的形式写入生成vendor/composer/autoload_namespaces.php这个文件中;

	在 psr-0 中,php 会将filename 中的`_`转义为`\`,这是因为在 psr-0 诞生之前, php 命名空间这个特性还没有出现;

3. psr-4

	```
	"autoload": {
		"psr-4": {
			"Screw\\":"src/Screw",
			"ScrewTest\\":"test/Screw"
		}
	}
	```
	对应文件目录为:

	```
	└── screw
	    ├── LICENSE
	    ├── README.md
	    ├── composer.json
	    ├── phpunit.xml.dist
	    └── src
	        ├── Screw
	        │   ├── Base64UrlSafe.php
	        │   ├── HashMap.php
	        │   ├── ObjectLib.php
	        │   └── Str.php
	        └── helper.php
	```

	当自动加载`Screw\HashMap`这个 class 时,会自动寻找并加载`src/Screw/HashMap.php`这个文件;psr-4 的配置会换成`namespace`为 key,dir path 为 value的 Map 形式写入到`vendor/composer/autoload_psr4.php`文件中;

	可以看到, 把 `Screw` 指向 `src` 之后psr-4就会默认将`src` 下面的class都有了`Screw`基本 namespace,而且 psr-4不会将`-`转义成`\`,因此文件结构不会像psr-0那么深.

4. files

	```
	"autoload": {
        "files": ["src/functions_include.php"],
    },
	```
	直接引入单个文件

### PSR-0 与 PSR-4 区别

1. 概念

	虽然 PSR-0 与 PSR-4 看上去大致是一样的,都是规范 PHP 自动加载的问题;

	PSR-4规范了如何指定文件路径从而自动加载类定义，同时规范了自动加载文件的位置。

	这个乍一看和PSR-0重复了，实际上，在功能上确实有所重复。

	区别在于PSR-4的规范比较干净，去除了兼容PHP 5.3以前版本的内容，有一点PSR-0升级版的感觉。

	当然，PSR-4也不是要完全替代PSR-0，而是在必要的时候补充PSR-0——当然，如果你愿意，PSR-4也可以替代PSR-0。

	PSR-4可以和包括PSR-0在内的其他自动加载机制共同使用。

	**PSR-4和PSR-0最大的区别是对下划线（underscore)的定义不同。PSR-4中，在类名中使用下划线没有任何特殊含义。而PSR-0则规定类名中的下划线_会被转化成目录分隔符。**

2. 代码样例

	composer使用 PSR-0风格:

		vendor/
			vendor_name/
				package_name/
					src/
						Vendor_Name/
							Package_Name/
								ClassName.php       
								# Vendor_Name\Package_Name\ClassName
					tests/
						Vendor_Name/
							Package_Name/
								ClassNameTest.php   
								# Vendor_Name\Package_Name\ClassName

	compser 使用 psr-4风格:

		vendor/
	   		vendor_name/
	        	package_name/
	            	src/
	                	ClassName.php       
	                	# Vendor_Name\Package_Name\ClassName
	            	tests/
	                	ClassNameTest.php   
	                	# Vendor_Name\Package_Name\ClassNameTest

	通过以上代码可以看出,psr-4有着更加简单的文件结构.

### 相关概念
1. [PHP 命名空间](http://php.net/manual/zh/language.namespaces.rationale.php)
2. [PSR-0](http://www.php-fig.org/psr/psr-0/)
3. [PSR-4](http://www.php-fig.org/psr/psr-4/)
4. [PSR-4自动加载规范](https://jifei.gitbooks.io/php-fig-standards/content/PSR-4-autoloader.html)

###  相关代码

```
<?php
/**
 * 一个具体项目的实例
 *
 * 当使用 SPL 注册此自动加载器后，执行以下语句将从
 * /path/to/project/src/Baz/Qux.php 载入 \Foo\Bar\Baz\Qux 类：
 *
 *      new \Foo\Bar\Baz\Qux;
 *      
 * @param string $class 完整的类名
 * @return void
 */
spl_autoload_register(function ($class) {

    // 具体项目命名空间前缀
    $prefix = 'Foo\\Bar\\';

    // 命名空间前缀的基目录
    $base_dir = __DIR__ . '/src/';

    // 判断类名是否具有本命名空间前缀
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        // 不含本命名空间前缀，退出本自动载入器
        return;
    }

    // 截取相应类名
    $relative_class = substr($class, $len);

    // 将命名空间前缀替作为文件基目录，然后
    // 将类名中的命名空间分隔符替换成文件分隔符,
    // 最后添加 .php 后缀
    $file = $base_dir . str_replace('\\', '/', $relative_class) . '.php';

    // 如果以上文件存在，则将其载入
    if (file_exists($file)) {
        require $file;
    }
});

```
