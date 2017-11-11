# tinyhttpd学习总结
## 总结
1. 知识点总结
* 多线程应用
if (pthread_create(&newthread , NULL, accept_request, client_sock) != 0)  
            perror("pthread_create"); 
* 进程间通信
进程：
 if ( (pid = fork()) < 0 ) {
  cannot_execute(client);
  return;
 }
 if (pid == 0)  /* child: CGI script */
 {
 } else {    /* parent */
}
管道和重定向：
/*建立管道*/  STDOUT 0 STDOUT 1 STDERR 2
    if (pipe(cgi_input) < 0) {
/* 把 STDOUT 重定向到 cgi_output 的写入端 */  
dup2(cgi_output[1], 1);  
/* 把 STDIN 重定向到 cgi_input 的读取端 */  
dup2(cgi_input[0], 0); 
设置环境变量：
// 实现了将变量传递给脚本
sprintf(query_env, "QUERY_STRING=%s", query_string);  
            putenv(query_env); 
 /*用 execl 运行 cgi 程序*/  
        execl(path, path, NULL); 
* 文件属性 
可以知道是否是可执行脚本
struct stat st; 
stat(path, &st)
if ((st.st_mode & S_IXUSR) || (st.st_mode & S_IXGRP) || (st.st_mode & S_IXOTH)    ) 
* execl 
1. 前后进程ID未改变，所以我们可以向该ID发送消息，控制一个不相关的可执行文件。
2. 当前进程的正文都被替换了，那么execl后的语句，即便execl退出了，都不会被执行。

2. 工作流程简述
* 工作流程：服务端启动端口监听，接收到请求。接收到请求后派生一个线程处理
if (pthread_create(&newthread , NULL, accept_request, client_sock) != 0) 。
如果是动态请求，需要cgi处理，会启动一个进程处理脚本（脚本和客户端的信息传递
靠父进程和cgi程序的管道（pipe(cgi_input)）通信）。所以当请求很多事会有很多进程
显然是很不好的。否则，直接在线程里面处理和读取文件返回。
* 常见问题： 
与cgi通信开启两个管道的原因在于，主进程和子进程读写的冲突；
父进程有通过管道写数据给cig, cgi脚本应该是靠标准输入介绍POST的数据


## 问题与思考


## 参考
1.很好的介绍了tinyhttpd 的工作方式和代码解析。
http://blog.csdn.net/jcjc918/article/details/42129311
2.源码下载地址:
http://sourceforge.net/projects/tinyhttpd/
3. http https 协议介绍  
https://www.cnblogs.com/EricaMIN1987_IT/p/3837436.html
https://www.cnblogs.com/binyue/p/4500578.html
