# MySQL 5.7 源码中关键方法和节点

## sql_parse.cc#do_command
所有mysql网络包信息的处理进口方法  
在方法中设置一般性的各种信息，包括连接超时时间等
该方法并无太多的信息量，但是作为所有命令的入口还是极好的
值得注意的是在这个方法中会对于网络数据进行解析，和具体的协议类型进行匹配
```
 rc= thd->get_protocol()->get_command(&com_data, &command);
```


##  sql_parse.cc#dispatch_command
真正开始处理网络包的入口方法  
在方法头部会对于贯穿整个查询过程的```THD *thd``` 对象进行初始化，后续这个对象将会是类似一个session的概念，承载整个查询过程中的关键信息  
之后整个方法的主要任务会按照网络包的协议类型进行不同分支的流程分发,一般情况下可以将一般的SQL执行的开始点设置到这个方法
```
 switch (command) 
```
## sql_parse.cc#mysql_parse
协议包COM_QUERY的标准处理方法
主要处理了如下几个模块的任务：
+ 解析SQL
+ 设置各种附加系统(processlist/binlog/general_log)等等
+ 在方法mysql_execute_command正式执行SQL


## sql_lex.cc#MYSQLlex
真正的SQL解析方法，具体的调用栈如下：
```
	mysqld.exe!MYSQLlex(YYSTYPE * yylval, YYLTYPE * yylloc, THD * thd) 行 1288	C++
 	mysqld.exe!MYSQLparse(THD * YYTHD) 行 19872	C++
 	mysqld.exe!parse_sql(THD * thd, Parser_state * parser_state, Object_creation_ctx * creation_ctx) 行 7089	C++
 	mysqld.exe!mysql_parse(THD * thd, Parser_state * parser_state) 行 5454	C++
```

## sql_parse.cc#mysql_execute_command
sql真正执行的方法,中间包含了sql执行的每个大体上的细节步骤
其中在进行一大堆前期准备之后,通过方法
```
  switch (lex->sql_command) 
```
对于SQL的类型进行区分，使得不同的SQL类型可以走到不同的方法中去


##  sql_parse.cc#execute_sqlcom_select
分化之后的select执行方法


## sql_select.cc#handle_query



## sql_executor#evaluate_join_record



## records.cc#init_read_record
READ_RECORD对象初始化方法  
READ_RECORD对象后续通过封装sort方法，对外提供完成排序的顺序记录，从而达成外部的数据有序使用的目的  
而本方法中则是对于READ_RECORD对象进行合理的初始化

+ 对于READ_RECORD对象进行初始化，清空内存内容
+ 根据打开table的属性设置以下的几个项目
     - info->record
     - info->ref_length
+ 根据表格的属性给info->read_record这个函数指针具体的几个函数类型和条件如下
     - 表格为临时表 & 使用插件字段？ & 插件字段打包？ ：```rr_unpack_from_tempfile<true>```
     - 表格为临时表 & 插件字段打包 ：```rr_unpack_from_tempfile<true>```
     - 其他表格为临时表的情况：```rr_from_tempfile```
     - 从cache中读取：```rr_from_cache```
     - 使用快速读取：```rr_quick```
     - table有在内存中的排序结果 & 不使用插件字段：```rr_from_pointers```
     - 不是以上所有情况：```rr_sequential```

总结：这个方法是根据输入的table的类型进行初始化READ_RECORD对象读取信息的方式，也就是对于后续read_record函数的封装。在java代码中往往会使用多态的方式进行实现，但是在C++中函数指针将会更加普遍的被使用，个人认为这不是一个太好的方法。