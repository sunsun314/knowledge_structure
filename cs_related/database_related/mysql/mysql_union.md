# mysql union不稳定逻辑调查


## union中的select执行路径

```
>	mysqld.exe!join_init_read_record(QEP_TAB * tab) 行 2460	C++
 	mysqld.exe!sub_select(JOIN * join, QEP_TAB * qep_tab, bool end_of_records) 行 1271	C++
 	mysqld.exe!do_select(JOIN * join) 行 944	C++
 	mysqld.exe!JOIN::exec() 行 199	C++
 	mysqld.exe!st_select_lex_unit::execute(THD * thd) 行 841	C++
 	mysqld.exe!handle_query(THD * thd, LEX * lex, Query_result * result, unsigned __int64 added_options, unsigned __int64 removed_options) 行 191	C++
 	mysqld.exe!execute_sqlcom_select(THD * thd, TABLE_LIST * all_tables) 行 5143	C++
 	mysqld.exe!mysql_execute_command(THD * thd, bool first_level) 行 2756	C++
 	mysqld.exe!mysql_parse(THD * thd, Parser_state * parser_state) 行 5559	C++
 	mysqld.exe!dispatch_command(THD * thd, const COM_DATA * com_data, enum_server_command command) 行 1429	C++
 	mysqld.exe!do_command(THD * thd) 行 995	C++
 	mysqld.exe!handle_connection(void * arg) 行 300	C++
 	mysqld.exe!pfs_spawn_thread(void * arg) 行 2190	C++
 	mysqld.exe!win_thread_start(void * p) 行 37	C
 	mysqld.exe!invoke_thread_procedure(unsigned int(*)(void *) procedure, void * const context) 行 92	C++
 	mysqld.exe!thread_start<unsigned int (__cdecl*)(void * __ptr64)>(void * const parameter) 行 115	C++
 	[外部代码]	

```