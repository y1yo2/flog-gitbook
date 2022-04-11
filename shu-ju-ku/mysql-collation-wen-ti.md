---
description: MySQL的CHARSET和Collation，是大家常遇到却不在意的属性（直到遇上生产问题）
---

# MySQL Collation问题

### Character set是什么？Collation是什么？

Character set，**字符集**是一组符号和编码，表示字符和编码的关系。例如，ASCII字符集，使用 1个字节即8位的空间进行编码，表示128个字符。

Collation，**校对规则**是一组比较**字符集**中字符的规则，比较字符的规则。

> A character set is a set of symbols and encodings.&#x20;
>
> A collation is a set of rules for comparing characters in a character set. Let's make the distinction clear with an example of an imaginary character set.

### MySQL版本对Character set, Collation的影响

MySQL 8.0前默认character set: latin1, collation: latin1\_swedish\_ci

MySQL 8.0后默认character set: utf8mb4, collation: utf8mb4\_0900\_ai\_ci

```sql
// 查看MySQL版本, Character set, Collation
SHOW VARIABLES LIKE '%version%';
 
SHOW VARIABLES LIKE 'character_set_server';
SHOW VARIABLES LIKE 'collation_server';
```

### MySQL如何使用Charset, Collation属性

1. 使用各种字符集存储字符串。
2. 使用各种排序规则比较字符串。
3. 在同一台服务器、同一个数据库甚至同一个表中**混合**具有不同字符集或排序规则的字符串。
4. 允许在**任何级别**指定字符集和排序规则。

```sql
// 设置数据库的charset, collation
// 设置表的charset, collation
// 设置字段的charset, collation
// 如果全都显示设置了，那么优先级顺序是 :
// SQL语句 > 列级别设置 > 表级别设置 > 库级别设置 > 实例级别设置。
ALTER DATABASE `basename` CHARACTER SET utf8 COLLATE utf8_bin;
ALTER TABLE `basename`.`tablename` COLLATE=utf8_bin;
ALTER TABLE `tablename` MODIFY COLUMN `name` varchar(8) CHARACTER SET utf8 COLLATE utf8_bin;

-- 查看字符集信息
SHOW CHARACTER SET;
SHOW CHARACTER SET LIKE 'utf8';

-- 查看所有校对规则
SHOW COLLATION;
SHOW COLLATION WHERE Charset = 'utf8';

-- 查询所有库的字符集与排列字符集
SELECT SCHEMA_NAME 'database', DEFAULT_CHARACTER_SET_NAME 'charset', DEFAULT_COLLATION_NAME 'collation' FROM information_schema.SCHEMATA;
```

### 具体的Charset, Collation

#### 关于Charset

utf8最多只支持3bytes长度的字符编码（MySQL中的utf8即utf8mb3），对于一些需要占据4bytes的文字（例如emoji需要4个字节，字符集不匹配会导致emoji存储错误），mysql的utf8不支持，要使用utf8mb4。

#### 关于Collation

ci 结尾是 case insensitive, 即 "大小写不敏感", a 和 A 会在字符判断中会被当做一样的，这样在需要判断大小时就不能满足要求。（例如无法判断字段的emoji重复）

cs 结尾(case sensitive区分大小写)。bin 结尾(binary二进制)。

例如常用的utf8mb4字符集对应的3个校对规则：`utf8mb4_general_ci`（默认）、`utf8mb4_unicode_ci`、`utf8mb4_bin`

`utf8mb4_bin`：直接将所有字符看作二进制串，然后从最高位往最低位比对。肯定区分大小写，且不存在特殊字符判断为相等。\
`utf8mb4_unicode_ci` 和 `utf8mb4_general_ci`对只使用中文和英文的系统无区别。但对需要国际化的系统（例如欧洲国家的字母），`utf8mb4_unicode_ci`会比`utf8mb4_general_ci`更符合当地语言。（例如，德语字母`“ß”`，在`utf8mb4_unicode_ci`中是等价于`"ss"`两个字母的（这是符合德国人习惯的做法），而在`utf8mb4_general_ci`中，它却和字母`“s”`等价。）

因此推荐使用 `utf8mb4_unicode_ci`，对于已经用了`utf8mb4_general_ci`的系统也可继续使用。需要区分大小写则使用 `utf8mb4_bin` 。\




> 参考文档：\
> [https://dev.mysql.com/doc/refman/8.0/en/charset-general.html](https://dev.mysql.com/doc/refman/8.0/en/charset-general.html)
