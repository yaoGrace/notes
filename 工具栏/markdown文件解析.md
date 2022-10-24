#### 功能介绍
> 使用PHP解析Markdown语法文件为html文件

#### 部署说明
>将 Parsedown.php 文件直接部署到 phpGrace/tools/ 文件夹下


#### 调用演示
```php
 //1. 创建实例
$Parsedown = new phpGrace\tools\Parsedown();
//2. 单独解析某一句
echo $Parsedown->text('Hello _Parsedown_!'); # 输出结果: <p>Hello <em>Parsedown</em>!</p>
// 也可以只解析内联元素
echo $Parsedown->line('Hello _Parsedown_!'); # 输出结果: Hello <em>Parsedown</em>!

// 3. 解析文件
$markdown = file_get_contents('statics/111.md');
$convertedHmtl = $Parsedown->setBreaksEnabled(true)->text($markdown);
// 4. 打印输出解析后的html
echo $convertedHmtl;

```