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

    import os,sys,paramiko
    from getpass import getpass
    paramiko.util.log_to_file('auto_ssh.log',0)
    def parse_user(user,default_host,default_port):
        """给定名字[@host[:port]],返回用户名，主机，端口
        必要时给出默认的主机和端口
        """
        if '@' not in user:
            return user,default_host,default_port
        user,host=user.split('@',1)
        if ':' in host:
            host,port=host.split(':',1)
        else
            port=default_port
        return user,host,port
    def autoSsh(users,cmds,host='localhost',port=22,timeout=5.0,
        maxsize=2000,passwords=None):
        """使用给定的或是默认的主机，端口以及超时，为每个用户运行命令
        将各个给定的命令以及它们的回应（每个回应不超过"maxsize"个字符
        打印到标准输出"""
        if passwords is None:
            passwords = {}
        for user in users:
            if user not in passwords:
                passwords[user]=getpass("Enter user '%s' password: " % user)

        for user in users:
            user,host,port = parse_user(user,host,port)
            try:
                transport=paramiko.Transport((host,port))
                transport.connect(username=user,password=passwords[user])
                channel=transport.open_session()
                if timeout:channel.settimeout(timeout)
                for cmd in cmds:
                    channel.exec_command(cmd)
                    response=channel.recv(maxsize)
                    print ' CMD %r(%r) -- %s' % (cmd,user,response) 
            except Excetion,err:
                print "ERR: unable to process %r: %s" % 9user,err
    if __name__=='__main__':
        logname=os.environ.get('LOGNAME',os.environ.get('USERNAME')
        host='localhost'
        port='22'
        usage="""
            usage: %s [-h host] [-p port] [-f cmdfile] [-c "command"] user1 user2..
            -c command
            -f command file
            -h default host (default:localhost)
            -p default port (default:22)
        Example: %s -c "echo $HOME" %s
        same as: %s -c "echo $HOME" %s@localhost:22""" % (sys.argv[0],
            sys.argv[0],logname,sys.argv[0],logname)
        import getopt
        optlist,userlist=getopt.getopt(sys.argv[1:],"c:f:h:p:")
        if not user_list:
            print usage
            sys.exit(1)
        cmd_list = []
        for opt,optarg in optlist:
            if opt== '-f':
                for r in open(optargs,'rU'):
                    if r.rstrip():
                        cmd_list.append(r)
            elif opt == '-c':
                command = optarg
                if command[0] == '"' and command[-1] = '"':
                    command=command[1:-1]
                cmd_list.append(command)
            elif opt ='-h':
                host=optarg
            elif opt='-p':
                port=optarg
            else:
                print 'unknow option %r' % opt
                print usage
                sys.exit(1)
        autoSsh(user_list,cmd_list,host=host,port=port)


### 通过HTTPS验证一个SSL客户端

    import httplib
    CERT_FILE='/home/robr/mycert'
    PKEY_FILE='/home/robr/mycert'
    HOSTNAME='localhost'
    conn=httplib.HTTPSConnection(HOSTNAME,key_file=PKEY_FILE,cert_file=CERT_FILE)
    conn.putrequest('GET','/ssltest')
    conn.endheaders()
    response=conn.getresponse()
    print response.read()
                

