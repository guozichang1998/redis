# redis

##命令处理流程

###客户端
- 对输入命令的处理
    - redis-cli.c/repl
        - redis-cli.c/cliSendCommand
            - hiredis.c/redisAppendCommandArgv
                - hiredis.c/redisFormatCommandArgv
                ```命令格式化```
                - hiredis.c/__redisAppendCommand    
                ```写入全局的 redisContext 的 write 缓冲区 obuf 中```
                                
- 向服务器发送命令
    - redis-cli.c/cliSendCommand
        - redis-cli.c/cliReadReply
            - hiredis.c/redisGetReply
                - hiredis.c/redisBufferWrite

- 获取服务器回复
    - redis-cli.c/cliSendCommand
        - redis-cli.c/cliReadReply
            - hiredis.c/redisGetReply
                - hiredis.c/redisBufferRead
                - hiredis.c/redisGetReplyFromReader
            - redis-cli.c/cliFormatReplyRaw
            - fwrite
            
 ###服务端
 - redis.c/initServer
    - redis.c/listenToPort
        - anet.c/anetTcpServer
            - anet.c/_anetTcpServer
            ```C
            // 建立socket套接字
            s = socket(p->ai_family,p->ai_socktype,p->ai_protocol))
            ```
    - redis.c/aeCreateFileEvent(,acceptTcpHandler,)
        - networking.c/acceptTcpHandler
            - anet.c/anetTcpAccept
            - networking.c/acceptCommonHandler
                - networking.c/createClient
                    - networking.c/readQueryFromClient
                    - ae.c/aeCreateFileEvent(server.el,fd,AE_READABLE,readQueryFromClient, c)
                    ```markdown
                    绑定readQueryFromClient到事件loop,redis的监听socket收到数据时将调用该函数进行处理
                    ```
 ###命令同步
 - redis.c/call
    - redis.c/propagate
        - replication.c/replicationFeedSlaves
            - networking.c/addReply
                - networking.c/prepareClientToWrite
                    - networking.c/sendReplyToClient