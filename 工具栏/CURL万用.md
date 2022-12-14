#### curl通讯类功能介绍
>curl通讯类，包含 get、post 方式 ，此类封装了常用的操作 基本通用。


#### 部署说明
>将 curl.php 文件直接部署到 phpGrace/tools/ 文件夹下
```php
//创建实例
$curl = new phpGrace\tools\curl(); 
```
> **使用演示**
#### GET方式
```php
$res  = $curl->get('http://api.hcoder.net');
//返回结果
echo $res; 
```

#### POST方式
```php
//post 数据
$data = array('name' => 'grace', 'age' => 10);
$res  = $curl->post('http://api.hcoder.net', $data);
//curl 状态
p($curl->http_status);
//传输时间毫秒
echo $curl->speed;
//返回结果
echo $res; 
```

#### 获取 curl 资源对象
> curl 资源对象保存在 phpGrace curl对象的 curlHandle 属性，您可以获取它并继续完成其他功能：
```php
$curl = $curl->curlHandle;
``` 


#### 设置cookie
```php
$curl->set_Cookie='填写cookie';
```

#### 设置模拟访问来源Refer
```php
$curl->set_Refer="填写来路网址";
```

#### 设置模拟UseaAgent
```php
$curl->set_UA = '填写模拟的UA信息'; 
// 例如 User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
// 填写后面的值 
```

#### 是否获取301跳转地址
```php
//默认为false ,不获取 ；true 为获取
$curl->get_LoadUrl = true; 
```

#### 是否获取返回头header信息
```php
//默认为false ,不获取 ；true 为获取
$curl->get_Header = true;

# 注意：返回的信息里面包含请求头和网页信息
```

#### 手动设置请求头
```php
// 拼接 例如
$headerString =  'accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
accept-encoding: gzip, deflate, br
accept-language: zh-CN,zh;q=0.9
cache-control: max-age=0';
// 设置
$curl->set_Header = $headerString;
```

#### 使用文件上传功能
```php
# 文件上传功能，此功能未测试是否能成功
# 第一步：设置上传接口
$apiUrl = '此处填写文件上传的接口链接';
# 第二步: 设置上传的表单名称 和 文件地址
$data = array(
    // 表单名称 => 文件地址
    'file'=>new CURLFile(realpath("Curl.jpg")) 
);  
# 第三步: 通过POST方式提交
$result = $curl->post($apiUrl,$data); 
```

## curl源码
```php
<?php
/********************************************************
 * 万能curl通信类 
 * @version    
 *******************************************************/
namespace phpGrace\tools;
class curl {
    public $httpStatus;             #curl 状态
    public $curlHndle;              #获取 curl 资源对象
    public $speed;                  #传输时间毫秒
    public $timeOut = 60;           #超时时间
    public $set_Cookie;             #设置cookie
    public $set_Refer;              #模拟访问来源Refer
    public $set_UA;                 #模拟UseaAgent
    public $get_LoadUrl = false;    #是否获取301跳转地址，默认不获取
    public $get_Header = false;     #是否查看返回Header信息
    public $set_Header;             #设置请求头

    public function __construct(){
        $this->curlHandle = curl_init();
        curl_setopt($this->curlHandle, CURLOPT_TIMEOUT, $this->timeOut);
    }

    public function setopt($key, $val){
        curl_setopt($this->curlHandle, $key , $val);
    }

    public function get($url){
        curl_setopt($this->curlHandle, CURLOPT_URL            , $url);
        curl_setopt($this->curlHandle, CURLOPT_RETURNTRANSFER , true);
        curl_setopt($this->curlHandle, CURLOPT_SSL_VERIFYPEER , false);
        curl_setopt($this->curlHandle, CURLOPT_SSL_VERIFYHOST , false);
        curl_setopt($this->curlHandle, CURLOPT_ENCODING       , 'gzip,deflate');
        curl_setopt($this->curlHandle, CURLOPT_TIMEOUT        , $this->timeOut);
        # 设置cookie
        if (isset($this->set_Cookie)) {
            curl_setopt($this->curlHandle, CURLOPT_COOKIE, $this->set_Cookie);
        }    
        # 设置模拟访问来源Refer
        if (isset($this->set_Refer)) {
            curl_setopt($this->curlHandle, CURLOPT_REFERER, $this->set_Refer);
        }
        # 设置模拟UA 浏览器
        if(isset($this->set_UA)){
            // 手动设置UA
            curl_setopt($this->curlHandle, CURLOPT_USERAGENT, $this->set_UA);
        }else{
            //默认UA
            curl_setopt($this->curlHandle,CURLOPT_USERAGENT,"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36");
        }
        # 是否查看返回Header信息
        if ($this->get_Header) {
            curl_setopt($this->curlHandle, CURLOPT_HEADER, true);
        }
        # 设置请求头
        if(isset($this->set_Header)){
            curl_setopt($this->curlHandle, CURLOPT_HTTPHEADER, $this->set_Header);
        }

        $result =  curl_exec($this->curlHandle);
        $this->http_status = curl_getinfo($this->curlHandle);
        # 是否获取301跳转地址
        if ($this->get_LoadUrl) {
            if (isset($Headers['redirect_url'])) {
                $result = $Headers['redirect_url'];
            }
        }
        $this->speed = round($this->httpStatus['pretransfer_time']*1000, 2);
        return $result;
    }

    public function post($url, $data=null){
        //如果参数为数组，自动拼接生成URL参数字符串
        if (is_array($data)) {
            $data = http_build_query($data);
        }
        curl_setopt($this->curlHandle, CURLOPT_POST, 1);
        curl_setopt($this->curlHandle, CURLOPT_POSTFIELDS, $data);
        return $this->get($url);
    }
}
```