---
title: mysql 存储过程异常处理
date: 2018-09-25 21:22:50
tags: 
- 存储过程
- mysql
categories: 总结
---


当我们处理存储过程的时候,难免会遇到如何将存储过程中的报错统一处理之后返回出来。在工作中正好需要用到这方面的知识，所以来学习并做个记录。

<!-- more -->

```
create procedure duplicate_teams(out p int)
BEGIN
set p = 1;
insert into test values (1,2);
set p =2;
END

```

这里的test表只有一个字段,我们临时测试使用,因此没有在对其创建表。
在进行调用时:

```
call pr_test(@q);
select @q;
```

在默认情况下，当存储过程运行出错时，过程会立即终止，并打印系统错误消息：

```
1136 - Column count doesn't match value count at row 1
```

输出为空。

## 1. 定义异常处理
通过对异常的定义，我们可以对其作出相应的处理步骤。

DECLARE ... HANDLER语句来进行定义处理：

```

DECLARE handler_action HANDLER
    FOR condition_value [, condition_value] ...
    statement
# 定义操作的动作:
# CONTINUE 继续执行当前程序
# EXIT 当前程序终止(退出当前declare所在的begin end)；
handler_action:
    CONTINUE
    | EXIT

# condition_value用以指明触发条件
condition_value:
    mysql_error_code
    | SQLSTATE [VALUE] sqlstate_value
    | condition_name
    | SQLWARNING
    | NOT FOUND
    | SQLEXCEPTION

# statement触发的时候需要做的操作
statement
```

## 2. 实践
将上述代码修改为:

```
create procedure duplicate_teams(out p int)
BEGIN

DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        set p = "10086";
    END;

set p = 1;
insert into test values (1,2);
set p =2;
END
```
此时再去调用得到结果为 2,当为EXIT模式时,则返回10086。


## 3. 获取错误内容

虽然我们获取了错误,但是并没有很方便的定位到报错位置以及报错信息。`mysql 5.6.4`以后的版本提供了`GET DIAGNOSTICS`来获取缓冲区的报错内容。


```
#关键字CURRENT表示从当前诊断区域检索信息。在MySQL中，它没有任何效果，因为这是默认行为。
GET [CURRENT] DIAGNOSTICS
{
    statement_information_item
    [, statement_information_item] ...
  | CONDITION condition_number
    condition_information_item
    [, condition_information_item] ...
}

# 
statement_information_item:
    target = statement_information_item_name

condition_information_item:
    target = condition_information_item_name

statement_information_item_name:
    NUMBER
  | ROW_COUNT

condition_information_item_name:
    CLASS_ORIGIN
  | SUBCLASS_ORIGIN
  | RETURNED_SQLSTATE
  | MESSAGE_TEXT
  | MYSQL_ERRNO
  | CONSTRAINT_CATALOG
  | CONSTRAINT_SCHEMA
  | CONSTRAINT_NAME
  | CATALOG_NAME
  | SCHEMA_NAME
  | TABLE_NAME
  | COLUMN_NAME
  | CURSOR_NAME



# 要获取语句信息，请将所需的语句项检索到目标变量中。此实例 GET DIAGNOSTICS将可用条件数和受行影响的计数分配给用户变量
GET DIAGNOSTICS @p1 = NUMBER, @p2 = ROW_COUNT;

#要获取条件信息，请指定条件编号并将所需的条件项检索到目标变量中。此实例GET DIAGNOSTICS将SQLSTATE值和错误消息分配给用户变量
GET DIAGNOSTICS CONDITION 1 @p3 = RETURNED_SQLSTATE, @p4 = MESSAGE_TEXT;

```


代码修正如下:

```
create procedure duplicate_teams(out result varchar(1024))
BEGIN
DECLARE code VARCHAR(1024);
DECLARE msg VARCHAR(1024);
DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
		GET DIAGNOSTICS CONDITION 1
        code = RETURNED_SQLSTATE, msg = MESSAGE_TEXT;
        SET result = CONCAT('insert failed, error = ',code,', message = ',msg);
    END;

set result = 1;
insert into test values (1,2);
set result =2;
END
```

输出为:

```
call pr_test(@q);
select @q;

@q
insert failed, error = 21S01, message = Column count doesn't match value count at row 1
```
## 4. 补充说明

### 关于错误编号

每个MySQL错误都有一个唯一的数字错误编号(`mysql_error_code`)，每个错误又对应一个5字符的SQLSTATE码(ANSI SQL 采用)。


SQLSTATE码对应的处理程序：

　　1、SQLWARNING处理程序：以‘01’开头的所有sqlstate码与之对应,对应为警告

　　2、NOT FOUND处理程序：以‘02’开头的所有sqlstate码与之对应为无数据

　　3、SQLEXCEPTION处理程序：不以‘01’或‘02’开头的所有sqlstate码，也就是所有未被SQLWARNING或NOT FOUND捕获的SQLSTATE(常遇到的MySQL错误就是非‘01’、‘02’开头的)

* 注意：‘01’、‘02’开头和‘1’、‘2’开头是有区别的，是不一样的错误sqlsate码。

