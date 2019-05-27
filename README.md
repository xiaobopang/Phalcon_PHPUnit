## 简介
 
 这是一个基于Phalcon框架的单元测试demo小教程，希望对初次编写单元测试的童鞋有所帮助。

 
### 前提

```
   1、在开始之前先假定你已经使用过Phalcon这个框架，对其有一定基础认知。
   2、其次你对composer包依赖管理有一定的了解
```

### 快速开始

#### 1.安装PHPUnit,以下为Linux操作环境：

````
        $ wget wget -O phpunit https://phar.phpunit.de/phpunit-8.phar

        $ sudo mv phpunit-8.phar /usr/local/bin/phpunit

        $ sudo chmod +x /usr/local/bin/phpunit

        $ phpunit --version

````
#### 如果输出结果如下，则表明你已经安装成功：

![执行结果](./test.png)

[PHPUnit](https://github.com/sebastianbergmann/phpunit) 👈点击左侧"PHPUnit"


#### 2、通过composer来引入相关测试组件，首先进入你的项目根目录执行一下命令：

```
        1）composer require --dev phpunit/phpunit ^8
        2）composer require phalcon/incubator
        3）以上安装结束后，在你的项目根目录下创建tests文件夹，并进入到tests文件夹下
        4）在tests文件夹下新建phpunit.xml，格式如下：

        <?xml version="1.0" encoding="UTF-8"?>
        <phpunit bootstrap="./TestHelper.php"
                backupGlobals="false"
                backupStaticAttributes="true"
                verbose="false"
                colors="true"
                cacheResult="true"
                convertErrorsToExceptions="true"
                convertNoticesToExceptions="true"
                convertWarningsToExceptions="true"
                mapTestClassNameToCoveredClassName="false"
                processIsolation="false"
                stopOnFailure="false"
                syntaxCheck="true">

            <testsuites>
                <testsuite name="Phalcon - Testsuite">
                    <!--仅当php版本不低于7.0的时候才执行单元测试-->
                    <directory suffix="Test.php" phpVersion="7.0" phpVersionOperator=">=">./</directory>
                    <file phpVersion="7.0" phpVersionOperator=">=">./Test/UnitCaseTest.php</file>
                </testsuite>
            </testsuites>
            <!--定义PHP变量-->
            <php>
                    <includePath>.</includePath>
                    <get  name="name" value="jack"/>
                    <post name="username" value="jack"/>
                    <post name="password" value="9cbf8a4dcb8e30682b927f352d6559a0"/>
            </php>
            <!--代码覆盖率白名单-->
            <filter>
                <whitelist processUncoveredFilesFromWhitelist="true">
                    <directory suffix=".php">./</directory>
                    <file>./UnitCaseTest.php</file>
                    <exclude>
                        <directory suffix=".php">./</directory>
                        <file>./UnitCaseTest.php</file>
                    </exclude>
                </whitelist>
            </filter>
        </phpunit>
```

#### 5）紧接着在你的tests文件夹下新建一个TestHelper.php，其内容如下：

```
        <?php
            /*
            * Created Date: Sunday May 5th 2019
            * Author: Pangxiaobo
            * Last Modified: Sunday May 5th 2019 5:57:13 pm
            * Modified By: the developer formerly known as Pangxiaobo at <10846295@qq.com>
            * Copyright (c) 2019 Pangxiaobo
            */

            use Phalcon\DI;
            use Phalcon\DI\FactoryDefault;

            ini_set('display_errors', 1);
            error_reporting(E_ALL);

            define('ROOT_PATH', __DIR__);
            define('PATH_INCUBATOR', __DIR__ . '/../vendor/phalcon/incubator/');
            define('PATH_CONFIG', __DIR__ . '/../api/config/config.php');
            define('PATH_MODELS', __DIR__ . '/../api/models/');
            define('PATH_CONTROLLERS', __DIR__ . '/../api/controllers/');
            define('PATH_SERVICES', __DIR__ . '/../api/services/');

            set_include_path(
                ROOT_PATH . PATH_SEPARATOR . get_include_path()
            );

            // 使用autoloader加载应用中的类，autoload依赖可以在composer vendor目录下找到
            $loader = new \Phalcon\Loader();

            $loader->registerDirs(array(
                ROOT_PATH,
                PATH_CONFIG,
                PATH_MODELS,
                PATH_SERVICES,
                PATH_CONTROLLERS,
            ));

            $loader->registerNamespaces(array(
                'Phalcon' => PATH_INCUBATOR . 'Library/Phalcon/',
            ));

            $loader->register();
            $config = include PATH_CONFIG;
            $di = new FactoryDefault();
            Di::reset();

            //这里我们注入一些需要用到的service服务，例如：db
            $di->set('db', function () use ($config) {
                return new \Phalcon\Db\Adapter\Pdo\Mysql($config->database->toArray());
            }, true);

            Di::setDefault($di);
```

####  6）好了，我们在创建完TestHelper.php之后，在tests目录下接着新建一个UnitTestCase.php的文件：

```
        <?php
            /*
            * Created Date: Sunday May 5th 2019
            * Author: Pangxiaobo
            * Last Modified: Sunday May 5th 2019 6:07:08 pm
            * Modified By: the developer formerly known as Pangxiaobo at <10846295@qq.com>
            * Copyright (c) 2019 Pangxiaobo
            */

            use Phalcon\Di;
            use Phalcon\Di\FactoryDefault;
            use Phalcon\Mvc\Model\Manager as ModelsManager;
            use \Phalcon\Test\UnitTestCase as PhalconTestCase;
            use Phalcon\Cache\Backend\Redis as BackRedis;
            use Phalcon\Cache\Frontend\Data as FrontData;

            abstract class UnitTestCase extends PhalconTestCase
            {

            /**
            * @var \Voice\Cache
            */
                protected $_cache;

                /**
                * @var \Phalcon\Config
                */
                protected $_config;

                /**
                * @var bool
                */
                private $_loaded = false;

                public function setUp(Phalcon\DiInterface $di = null, Phalcon\Config $config = null)
                {
                    global $config;

                    if (is_null($config)) {
                        $this->_config = include __DIR__ . '/../api/config/config.php';
                    } else {
                        $this->_config = $config;
                    }
                    
                    // 在测试期间根据需要加载一些服务
                    $di = new FactoryDefault();
                    Di::reset();

                    $di->set('db', function () use ($config) {
                        return new \Phalcon\Db\Adapter\Pdo\Mysql($config->database->toArray());
                    });
                    
                    $di->set('modelsManager', function () {
                        return new ModelsManager();
                    }, true);

                    $di->set("config", function () use ($config) {
                        return $config;
                    });
                    
                    $di->setShared('serviceCache', function () use ($config) {
                        $frontCache = new FrontData([
                            'lifetime' => 2592000,
                        ]);
                        return new BackRedis($frontCache,(array) $config->redisCache);
                    });

                    Di::setDefault($di);
                    $this->setDi($di);

                // parent::setUp($di, $config);
                    $this->_loaded = true;
                }

                /**
                * 重置testcase环境
                *
                * @return void
                */
                protected function tearDown()
                {
                    $di = $this->getDI();
                    $di::reset();
                    parent::tearDown();
                }

                /**
                * Check if the test case is setup properly
                * @throws \PHPUnit_Framework_IncompleteTestError;
                */
                public function __destruct()
                {
                    if (!$this->_loaded) {
                        throw new \PHPUnit_Framework_IncompleteTestError('Please run parent::setUp().');
                    }
                }
            }
```

#### 7）为了使你的测试和你的应用隔离开来，避免污染整个项目源代码，我们可以为单元测试创建命名空间。好了，接下来我们需要在tests文件夹下接着创建一个名叫Test的文件夹。


#### 8）然后，我们在Test文件里新建一个名叫CaseTest.php的文件。其内容如下：

```
        <?php
            /*
            * Created Date: Monday May 6th 2019
            * Author: Pangxiaobo
            * Last Modified: Monday May 6th 2019 2:24:53 pm
            * Modified By: the developer formerly known as Pangxiaobo at <10846295@qq.com>
            * Copyright (c) 2019 Pangxiaobo
            */

            namespace Test;
            /**
            * Class UnitCaseTest
            */
            class CaseTest extends \UnitTestCase
            {
                /**
                * 控制器取值测试
                * @return void
                */
                public function testController()
                {
                    $logStatus = TestController::$logStatus;
                    $this->assertEquals($logStatus, 1, "获取到控制器的logStatus=" . $logStatus);
                }

                /**
                * 输出测试
                *
                * @return void
                */
                public function testHello()
                {
                    $name = $_GET["name"];
                    $test = new TestController();
                    $test->hello($name);
                    $this->expectOutputString('hello,jack');
                }

                /**
                * 模型类测试
                *
                * @return void
                */
                public function testModel()
                {
                    $testModel = new TestModel();
                    $testModel->source = "Test";
                    $testModel->listFields = ["id", "user", "age"];
                    $listInfo = $testModel->retrieve(0, 3);
                    // 读出的条目数量是否正确
                    $this->assertEquals(
                        count($listInfo["List"]),
                        3,
                        "查询数据和预期不符"
                    );
                }

                /**
                * 获取参数测试
                *
                * @return void
                */
                public function testTest()
                {
                    $username = $_POST['username'];
                    $password = $_POST['password'];
                    $this->assertEquals($username, 'jack', "用户名有误");
                    $this->assertEquals($password, '9cbf8a4dcb8e30682b927f352d6559a0', "密码有误");
                }
            }

```       

 #### 9）因为我们之前在UnitTestCase.php的setUp()中做了一些服务的依赖注入，以及在TestHelper.php中注册了相关的空间，并引入了Autoload相关文件。到此你就可以开心的编写你所想要的单元测试代码了。原文附上tests文件夹，仅供参考，如发现错误,还望指出。谢谢～
 
 #### 10）原文参考了Phalcon官网提供的相关单元测试文章，下面有相关链接：

```

[PhalconTestUnit](https://docs.phalconphp.com/3.4/en/unit-testing.html) 👈点击左侧"PhalconTestUnit"查看相关文章

