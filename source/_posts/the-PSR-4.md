title: PSR 规范 —— PSR-4
date: 2016-05-07 17:21:31
tags: [php]
---

# PHP Standard Recommendations

> PSR 是 PHP Standard Recommendations 的简写，它是由 [PHP FIG](https://github.com/php-fig) 组织制定的 PHP 规范，是 PHP 开发的实践标准。

## 序号 4：自动加载规范

本 PSR 是关于由文件路径 [自动加载](http://php.net/autoload) 对应类的相关规范，本规范是可互操作的，可以作为任一自动载入规范的补充，其中包括 PSR-0（已废弃），此外，本 PSR 还包括自动载入的类对应的文件存放路径规范。

### 关于“能愿动词”的使用

为了避免歧义，文档大量使用了“能愿动词”，对应的解释如下：
- `必须（MUST）`：绝对，严格遵循，请照做，无条件遵守；
- `一定不可（MUST NOT）`：禁令，严令禁止；
- `应该（SHOULD）`：强烈建议这样做，但是不强求；
- `不该（SHOULD NOT）`：强烈不建议这样做，但是不强求；
- `可以（MAY）` 和 `可选（OPTIONAL）`：选择性高一点，在这个文档内，此词语使用较少；

> 参见：[RFC 2119](http://www.ietf.org/rfc/rfc2119.txt)


### 详细说明

1.此处的‘类’泛指所有的‘class 类’、‘接口’、‘traits 可复用代码块’以及其它类似结构。
2.一个完整的类名需要具备以下结构

```php
 \<命名空间>(\<子命名空间>)*\<类名>
```
- 完整的类名 **必须** 要有一个顶级命名空间，被称为 'vendor namespace'
- 完整的类名 **可以** 有一个或多个子命名空间
- 完整的类名 **必须** 有一个最终的类名
- 完整的类名中任意一部分中的下划线都是没有特殊含义的
- 完整的类名 **可以** 由任意大小写字母组成
- 所有类名都 **必须** 是大小写敏感的。

3.当根据完整的类名载入相应的文件
- 完整的类名中，去掉最前面的命名空间分隔符，前面连续的一个或多个命名空间和子命名空间，作为‘命名空间前缀’，其必须与至少一个‘文件基目录’相对应。
- 紧接命名空间前缀后的子命名空间 **必须** 与相应的‘文件基目录’相匹配，其命名空间分隔符将作为目录分隔符。
- 末尾的类名 **必须** 与对应的以 `.php` 为后缀的文件同名。
- 自动加载器（autoloader）的实现 **一定不可** 抛出异常、**一定不可** 触发任一级别的错误信息以及 **不应该** 有返回值。

### 例子

下表展示了符合规范完整类名、命名空间前缀和文件基目录所对应的文件路径。

完整类名                     | 命名空间前缀    | 文件基目录             | 文件路径
---                          | ---             | ---                    | ---
\Acme\Log\Writer\File_Writer | Acme\Log\Writer | ./acme-log-writer/lib/ | ./acme-log-writer/lib/File_Writer.php
\Aura\Web\Response\Status    | Aura\Web        | /path/to/aura-web/src/ | /path/to/aura-web/src/Response/Status.php
\Symfony\Core\Request        | Symfony\Core    | ./vendor/Symfony/Core/ | ./vendor/Symfony/Core/Request.php
\Zend\Acl                    | Zend            | /usr/includes/Zend/    | /usr/includes/Zend/Acl.php

关于本规范的实现，可参阅 [相关实例](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader-examples.md)

> 注意： 实例并 **不** 属于规范的一部分，且随时 **会** 有所变动。

来自：[https://psr.phphub.org](https//psr.phphub.org)