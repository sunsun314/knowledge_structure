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