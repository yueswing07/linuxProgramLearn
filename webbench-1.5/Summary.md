# webbench学习总结
## 总结
1. 知识点总结
* 主机名域名获取
// 将字符串转换为32位二进制网络字节序的IPv4地址
    inaddr = inet_addr(host);
    if (inaddr != INADDR_NONE)
        memcpy(&ad.sin_addr, &inaddr, sizeof(inaddr));
    else
    {
        // 使用域名或主机名获取ip地址
        hp = gethostbyname(host);
        if (hp == NULL)
            return -1;
        memcpy(&ad.sin_addr, hp->h_addr, hp->h_length);
    }
* 并发处理多进程
并发处理不用多线的原因在于一个进程能开启的线程数受限（<256).
 // 根据并发数创建子进程 
    for(i=0;i<clients;i++)
    {
        pid=fork();
        if(pid <= (pid_t) 0) // 这里很关键 避免了子进程在起子进程
        {
           /* child process or error*/
            // 注意这里子进程sleep(1) 有效避免CUP过高
            sleep(1); /* make childs faster */
            break; 
        }
    }

* 执行时间控制  
定时通过信号机制实现，每个进程都有相同的定时。
实现： 设置一个全局变量timerexpired，启动定时，定时时间到时调用处理函数
设置timerexpired，在业务程序处理循环中不断判断timerexpired来控制是否跳出循环。
    struct sigaction sa;
    /* setup alarm signal handler */
    // 设置alarm定时器处理函数
    sa.sa_handler=alarm_handler;
    sa.sa_flags=0;
    // sigaction 成功则返回0，失败则返回-1
    // 超时会产生信号SIGALRM，用sa中的指定函数处理
    if(sigaction(SIGALRM,&sa,NULL)) 
        exit(3);
    alarm(benchtime); // 开始计时
// timerexpired = 1 时，定时结束
static void alarm_handler(int signal)
{
    timerexpired=1;
}   
* getopt_long 命令行参数解析
* 管道使用
fdopen()函数：将文件描述词转为文件指针，方便了读写
fscanf(f,"%d %d %d",&i,&j,&k);
webbench中在多个进程进行写管道的情况下，在代码中没有采取同步措施，程序是如何保持数据正确呢？
管道写的数据不大于 PIPE_BUF 时，系统可以保证写的原子性。

2. 工作流程简述
* 工作流程：说一下程序执行的主要流程：
      1. 解析命令行参数，根据命令行指定参数设定变量，可以认为是初始化配置。
      2.根据指定的配置构造 HTTP 请求报文格式。
      3.开始执行 bench 函数，先进行一次 socket 连接建立与断开，测试是否可以正常访问。
      4.建立管道，派生根据指定进程数派生子进程。
      5.每个子进程调用 benchcore 函数，先通过 sigaction 安装信号，用 alarm 设置闹钟函数，接着不断建立 socket 进行通信，与服务器交互数据，直到收到信号结束访问测试。子进程将访问测试结果写进管道。
      6.父进程读取管道数据，汇总子进程信息，收到所有子进程消息后，输出汇总信息，结束。。
* 常见问题： 
与cgi通信开启两个管道的原因在于，主进程和子进程读写的冲突；
父进程有通过管道写数据给cig, cgi脚本应该是靠标准输入介绍POST的数据


## 问题与思考

## 参考
1.
https://www.2cto.com/kf/201609/545003.html
很好的介绍了Webbench的工作方式和代码解析。
http://blog.csdn.net/jcjc918/article/details/44965951
https://github.com/imyouxia/Source/blob/master/webbench-1.5/webbench.c
2.
源码下载地址:
http://home.tiscali.cz/~cz210552/webbench.html