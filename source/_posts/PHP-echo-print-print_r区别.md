---
title: PHP从本质上分析echo、print、print_r三者之间的区别？
date: 2019-08-21 09:38:25
categories:
- PHP
tags:
- PHP
- 源码分析
---

`echo`、`print`、`print_r`在PHP开发中都是常见的，但每次只管用，不理解它们之前的区别，包括在很多的面试中都会提及三者之间的区别。我们都知道它们都可以输出并打印字符串，但本质上它们有什么区别呢？

<!-- more -->

## echo

PHP官方文档([echo](https://www.php.net/manual/zh/function.echo.php))说明：
> echo ( string $arg1 [, string $... ] ) : void
>
> echo 不是一个函数（它是一个语言结构）， 因此你不一定要使用小括号来指明参数，单引号，双引号都可以。 echo （不像其他语言构造）不表现得像一个函数， 所以不能总是使用一个函数的上下文。 另外，如果你想给echo 传递多个参数， 那么就不能使用小括号。
>
> 和 print 最主要的不同之处是， echo 接受参数列表，并且没有返回值。

从上面的说明可以看出echo是语言结构而不是函数。那么什么是语言结构呢？说简单的就是PHP的关键词，语法的一部分。而函数由代码块组成，从源码的角度说，也就是基于Zend引擎的基础来实现的。

### 词法解析&语法分析

PHP是脚本语言，因此所有的符号都会先经过词法解析和语法分析阶段，这两个阶段分别由`lex`和`yacc`完成。分别对应PHP源码中的`Zend/zend_language_scanner.l`和`Zend/zend_language_parser.y`。

```
<ST_IN_SCRIPTING>"echo" {
	RETURN_TOKEN(T_ECHO);
}
```

```
token:
%token T_ECHO       "echo (T_ECHO)"

statement:
...
T_ECHO echo_expr_list ';'		{ $$ = $2; }
...

echo_expr_list:
		echo_expr_list ',' echo_expr { $$ = zend_ast_list_add($1, $3); }
	|	echo_expr { $$ = zend_ast_create_list(1, ZEND_AST_STMT_LIST, $1); }
;

echo_expr:
	expr { $$ = zend_ast_create(ZEND_AST_ECHO, $1); }
;

expr:
		variable					{ $$ = $1; }
	|	expr_without_variable		{ $$ = $1; }
;
```

PHP语句先执行词法分析得到TOKEN，然后通过语法分析关联TOKEN得到抽象语法树（AST）。具体的输出我还没弄明白，暂时先写这么多。

详细请参考[深入理解PHP之echo](https://juejin.im/post/5b60648e6fb9a04fcd586af1#heading-9)

## print_r

PHP官方文档([print_r](https://www.php.net/manual/zh/function.print-r.php))说明：
> print_r ( mixed $expression [, bool $return = FALSE ] ) : mixed
>
> print_r() 以人类易读的格式显示一个变量的信息。
>
> print_r()、 var_dump()、 var_export() 都会显示对象 protected 和 private 的属性。 Class 的静态属性（static） 则不会显示。

print_r是PHP的一个函数，有返回值。
