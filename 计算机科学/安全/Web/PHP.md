一种颇具<ruby>风味<rt>漏洞</rt></ruby>的，文法比较「古典」的**脚本**语言。

# 内联格式
其可以直接插入html等地方。格式：
```php
<?php  
echo "Hello World!";  
?>
```
# 文法 (Syntax)
形似**C**，大部分语句形似如下：
```php
ciallo(arg0, arg1);
```
除少数例外如`echo`:
```php
echo "Ciallo";
```
变量前有`$`标记：
```php
$greetings="Ciallo"
```
没有声明，弱类型，类型分为以下几种。
- String（字符串）
- Integer（整型）
- Float（浮点型）
- Boolean（布尔型）
- Array（数组）
- Object（对象）
- NULL（空值）
- Resource（资源类型）

#### **[[弱比较]]**

#### **[[(反)序列化]]**
