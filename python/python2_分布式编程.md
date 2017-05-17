分布式编程(python2)
===========================
### 实现一个XML-RPC方法调用

    from xmlrpclib import Server
    server=Server("http://www.oreillynet.com/meerkat/xml-rpc/server.php")
    print server.meerkat.getItems(
        {'search':'[Pp]ython','num_itmes':5,'descriptions':0}
    )


### 服务XML-RPC请求
* 服务端

    ```
    import SimpleXMLRPCServer
    class StringFunctions(object):
        def __init__(self):
            #使用python标准库的string模块
            import string
            self.python_string=string
        def _privateFunction(self):
            #这个函数无法透过XML-RPC直接调用，因为它以
            #一个下划字符开始，它是私有的
            return "you'll never get this result on the client"
        def chop_in_half(self,astr):
            return astr[:len(astr)/2]
        def repeat(self,astr,times):
            return astr * times
    if __name__=='__main__':
        server=SimpleXMLRPCServer.SimpleXMLRPCServer(('localhost',8000))
        server.register_instance(StringFunctions())
        server.register_function(lambda astr:'_'+astr,'_string')
        server.serve_forever()
    ```
* 客户端调用
    
    ```
    import xmlrpclib
    server=xmlrpclib.Server("http://localhost:8000")
    print server.chop_in_half("I am a confident guy")
    #输出:I am a con
    print server.repeat("Repeation is the key to learning!\n",5)
    #输出5行，
    print server._string("<=underscore")
    #输出_<=underscore
    print server.python_string.join(['I','like it!']," don't "]
    #输出 I don't like it!
    print server._privateFunction() #这会抛出异常
    ```

### 允许XML-RPC服务被远程终止

    import SimpleXMLRPCServer
    runing = True
    def finis():
        global running
        running = False
        return 1

    #函数式写法
    server=SimpleXMLRPCServer.SimpleXMLRPCServer(('localhost',8000))
    server.register_function(finis)
    while running:
        server.handle_request()

    #子类化写法
    class MyServer(SimpleXMLRPCServer.SimpleXMLRPCServer):
        def server_forever(self):
            while running:
                self.handle_request()
    server=MyServer(('127.0.0.1',8000))
    server.register_function(finis)
    server.server_forever()


### SimpleXMLRPCServer的一些细节

    from SimpleXMLRPCServer import SimpleXMLRPCServer as BaseServer
    class Server(BaseServer):
        def __init__(self,host,prot):
            BaseServer.__init__(self,(host,port))
        def server_bind(self):
            #在服务被杀掉后允许迅速重启服务
            import socket
            self.sock.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)
            BaseServer.server_bind(self)
        allowedClientHosts='127.0.0.1','192.168.0.15',
        def verify_request(self,request,client_address):
            #除了特定的主机，禁止其他计算机的访问
            return client_address[0] in self.allowedClientHosts

### 使用SSH执行远程登录


