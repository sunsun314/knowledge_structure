# mysql 网络相关模块调查记录

## 登录过程中的调查记录

### 背景
在mysql登录过程中，如果client一段时间没能收到greeting包则直接会报错'登录'异常，需要根据源码的现象调查为啥出现这个，并且给出解释

### 代码模块调查经过
通过DEBUG进行mysql客户端部分的源码调查

client端口的网络包(登录OK返回包)收取或者等待堆栈如下所示：
```
>	mysql.exe!inline_mysql_socket_recv(const char * src_file, unsigned int src_line, st_mysql_socket mysql_socket, char * buf, unsigned __int64 n, int flags) 行 823	C
 	mysql.exe!vio_read(st_vio * vio, unsigned char * buf, unsigned __int64 size) 行 123	C
 	mysql.exe!vio_read_buff(st_vio * vio, unsigned char * buf, unsigned __int64 size) 行 166	C
 	mysql.exe!net_read_raw_loop(st_net * net, unsigned __int64 count) 行 672	C++
 	mysql.exe!net_read_packet_header(st_net * net) 行 762	C++
 	mysql.exe!net_read_packet(st_net * net, unsigned __int64 * complen) 行 822	C++
 	mysql.exe!my_net_read(st_net * net) 行 899	C++
 	mysql.exe!cli_safe_read_with_ok(st_mysql * mysql, char parse_ok, char * is_data_packet) 行 1040	C
 	mysql.exe!cli_safe_read(st_mysql * mysql, char * is_data_packet) 行 1174	C
 	mysql.exe!cli_read_change_user_result(st_mysql * mysql) 行 2784	C
 	mysql.exe!run_plugin_auth(st_mysql * mysql, char * data, unsigned int data_len, const char * data_plugin, const char * db) 行 4018	C
 	mysql.exe!mysql_real_connect(st_mysql * mysql, const char * host, const char * user, const char * passwd, const char * db, unsigned int port, const char * unix_socket, unsigned long client_flag) 行 4767	C
 	mysql.exe!sql_real_connect(char * host, char * database, char * user, char * password, unsigned int silent) 行 4979	C++
 	mysql.exe!sql_connect(char * host, char * database, char * user, char * password, unsigned int silent) 行 5122	C++
 	mysql.exe!main(int argc, char * * argv) 行 1323	C++
 	[外部代码]	

```
其中read_change_user_result为调用函数的函数指针，通过调用这个函数进行数据的等待

从这个堆栈中我们可以分析得到如下的具体网络调用的堆栈(其中大部分内容为同步获取)：
```
my_net_read
(抽象网络读取)
   |
     --net_read_packet 
		(mysql协议网络包读取)
            |
             -- net_read_packet_header 
				 (按照mysql网络包的协议特点优先读取包头)
                |
                --net_read_raw_loop
				    (循环读取网络数据，输入参数读取的网络信息字节数count)
						|
						 --vio_read/vio_read_buff函数指针决定 
						    (正真的网络层从这里开始，封装了不同操作环境下的网络实现)
							  |
							   -- mysql_socket_recv(socket,buf,size,flag)
							          通过宏定义在不同编译环境下函数具有不同的实现
									   |
									    --recv DEBUG环境下windows提供的网络信息接收方法

```



### load data file数据获取堆栈
```
>	mysqld.exe!vio_read(st_vio * vio, unsigned char * buf, unsigned __int64 size) 行 123	C
 	mysqld.exe!net_read_raw_loop(st_net * net, unsigned __int64 count) 行 672	C++
 	mysqld.exe!net_read_packet_header(st_net * net) 行 756	C++
 	mysqld.exe!net_read_packet(st_net * net, unsigned __int64 * complen) 行 822	C++
 	mysqld.exe!my_net_read(st_net * net) 行 899	C++
 	mysqld.exe!_my_b_net_read(st_io_cache * info, unsigned char * Buffer, unsigned __int64 Count) 行 59	C++
 	mysqld.exe!_my_b_get(st_io_cache * info) 行 1291	C
 	mysqld.exe!READ_INFO::read_field() 行 1619	C++
 	mysqld.exe!read_sep_field(THD * thd, COPY_INFO & info, TABLE_LIST * table_list, List<Item> & fields_vars, List<Item> & set_fields, List<Item> & set_values, READ_INFO & read_info, const String & enclosed, unsigned long skip_lines) 行 1058	C++
 	mysqld.exe!mysql_load(THD * thd, sql_exchange * ex, TABLE_LIST * table_list, List<Item> & fields_vars, List<Item> & set_fields, List<Item> & set_values, enum_duplicates handle_duplicates, bool read_file_from_client) 行 561	C++
 	mysqld.exe!mysql_execute_command(THD * thd, bool first_level) 行 3647	C++
 	mysqld.exe!mysql_parse(THD * thd, Parser_state * parser_state) 行 5559	C++
 	mysqld.exe!dispatch_command(THD * thd, const COM_DATA * com_data, enum_server_command command) 行 1509	C++
 	mysqld.exe!do_command(THD * thd) 行 995	C++
 	mysqld.exe!handle_connection(void * arg) 行 300	C++
 	mysqld.exe!pfs_spawn_thread(void * arg) 行 2190	C++
 	mysqld.exe!win_thread_start(void * p) 行 37	C
 	mysqld.exe!invoke_thread_procedure(unsigned int(*)(void *) procedure, void * const context) 行 92	C++
 	mysqld.exe!thread_start<unsigned int (__cdecl*)(void * __ptr64)>(void * const parameter) 行 115	C++
 	[外部代码]	

```