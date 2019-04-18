[TOC]





# 前言





# 一、引入`Thymeleaf`

通过引入`Thymeleaf`的命名空间，即可使用`Thymeleaf`

```html
<html xmlns:th="https://thymeleaf.org">  
```





# 二、`Thymeleaf`指令

## 1.标准表达式

Thymeleaf 提供了多种标准表达式包括：

- 简单表达式：
    - Variable expressions（变量表达式）`${...}`
    - Selection expressions（选择表达式）`*{...}`
    - Message (i18n) expressions（消息表达式） `#{...}`
    - Link (URL) expressions（链接表达式）`@{...}`
    - Fragment expressions（分段表达式）`~{...}`
- 字面量:
    - 文本：`'one text'`、`'Another one!'`等；
    - 数值：0、34、3.0、12.3等；
    - 布尔：true、false
    - Null：null
    - Literal token(字面标记): one、sometext、 main等；
- 文本操作:
    - 字符串拼接：`+`
    - 文本替换：`|The name is ${name}|`
- 算术操作:
    - 二元运算符：`+`、`-`、 `*`、`/`、`%`
    - 减号（单目运算符）：`-`
- 布尔操作：
    - 二元运算符：`and`、`or`
    - 布尔否定（一元运算符）：`!`、`not`
- 比较和等价：
    - 比较：`>`、`<`、`>=`、`<=`（`gt`、`lt`、`ge`、`le`）
    - 等价:`==`、`!=`（`eq`、`ne`）
- 条件运算符：
    - If-then：`(if) ? (then)`
    - If-then-else：`(if) ? (then) : (else)`
    - Default：`(value) ?: (defaultvalue)`
- 特殊标记：
    - No-Operation（无操作）：`_`



















# 参考资料

1. [https://github.com/waylau/thymeleaf-tutorial](https://github.com/waylau/thymeleaf-tutorial/blob/master/SUMMARY.md)
2. 