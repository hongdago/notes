网络编程(python2)
=============================
### 通过Socket数据报传输消息
* server端

    ```
    import socket
    port=8081
    s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
    #从给定的端口，从任何发送者，接收UDP数据包
    s.bind(("",port))
    print "waiting on port:",port
    while True:
        #接收一个数据报（最大到1024个字节）
        data,addr=s.recvfrom(1024)
        print "Received:",data,"from",addr
    ```
* client端
    
    ```
    import socket
    port=8081
    host="localhost"
    s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
    s.sendto("Holy Guido! It's working.",(host,port))

    #发送巨大的数据包消息msg
    BUFFER=1024
    while msg:
        bytes_sent=s.sendto(msg[:BUFFER],(host,port))
        msg=msg[bytes_sent:]
    ```

### 从Web抓取文档

    from urllib import urlopen
    doc=urlopen("http://www.python.org").read()
    print doc


### 过滤FTP站点列表

    import socket ,ftplib
    def isFTPSiteUp(site):
        try:
            frplib.FTP(site).quit()
        except socket.error:
            return False
        else:
            return True
    def filterFTPsites(sites):
        return [site for site in sites if isFTPSiteUp(site)]

### 通过SNTP协议从服务器获取时间
> 一个SNTP交换是以客户端发送一个以字节"\x1b"开头的长48字节的UDP数据报为开始的，
> 服务器的应答也是一个48字节的数据报，这个数据报由12个网络顺序的长字节(4字节）
> 组成。

    import socket,struct,sys,time
    TIME1970=2208988800L
    clinet=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
    data='\x1b'+'\0'*47
    client.sendto(data,(sys.argv[1],123))
    data,address=client.recvfrom(1024)
    if data:
        print 'Response received from:',address
        t=struct.unpack('!12I',data)[10]
        t-=TIME1970
        print '\tTime=%s' % time.ctime(t)

### 发送HTML邮件
> 发送一个HTML邮件并附上邮件内容的纯文本版本，这样邮件内容就能够被不支持HTML
> 的邮件客户端读取了。

    def createhtmlmail(subject,html,text=None):
        "创建MIME消息，最终呈现为HTML或文本"
        import MimeWrite,mimetools,cStringIO
        if text is None:
            #创建HTML字符串呈现的纯文本内容
            #除非通过参数指定了更好的纯文本版本
            import htmllib,formatter
            textout=cStringIO.StringIO()
            formtext=formatter.AbstractFormatter(formatter.DumbWrite(textout))
            parser=htmllib.HTMLParser(formtext)
            parser.feed(html)
            parser.close()
            text=textout.getvalue()
            del textout,formtext,parser
        out=cStringIO.StringIO() #消息的输出缓存
        htmlin = cStringIO.StringIO(html) #HTML的消息缓存
        textin = cStringIO.StringIO(text) #纯文本的消息缓存
        writer = MimeWrite.MimeWrite(out)
        #设置一些基本的头部，在此放入标题，根据RFC的规定
        #smtplib.sendmail会在消息中找标题
        writer.addheader("Subject",subject)
        writer.addheader("MIME-Version","1.0")
        #消息的多头部分，在某些邮件客户端中Multipart/alternatives
        #比multipart/mixed工作的更好
        writer.startmultipartbody('alternative')
        writer.flushheaders()
        #纯文本段:直接复制，假设为iso-8859-1
        subpart=writer.nextpart()
        pout=subpart.startbody("text/plain",[('charset','iso-8859-1')]
        pout.write(txtin.read())
        txtin.close()
        #消息的HTML部分，设为quoted-printable,以防万一
        subpart=writer.nextpart()
        subpart.addheader("Content-Transfer-Encoding","quoted-printable")
        pout=subpart.startbody("text/html",[('charset','us-ascii')])
        mimetools.encode(htmlin,pout,'quoted-printable)
        htmlin.close()
        #完工;关闭writer并将消息作为字符串返回
        writer.lastpart()
        msg=out.getvalue()
        out.close()
        return msg

### 在MIME消息中绑入文件

    import base64 ,quopri
    import mimetypes,email.Generator,email.Message
    import cStringIO,os
    #地址
    toAddr = "example@example.com"
    fromAddr="example@example.com"
    outputFile='dirContentsMail'
    def main():
        mainMsg=email.Message.Message()
        mainMsg["To"] = toAddr
        mainMsg["From"] = fromAddr
        mainMsg["Subject"] = "Directory contents"
        mainMsg["Mime-version"] = "1.0"
        mainMsg["Content-type"]="Multipart/mixed"
        mainMsg.preamble="Mime message\n"
        mainMsg.epilogue="" #确保消息以换行符为结束
        #获得文件名
        fileNames=[f for f in os.listdir(os.curdir) if os.path.isfile(f)]
        for fileName in fileNames:
            contentType,ignored = mimetypes.guess_type(fileName)
            if contentType is None: #如果猜不中
                contentType = "application/octet-stream"
            contentsEncoded = cStringIO.StringIO()
            f=open(filename,"rb")
            mainType=contentType[:contentType.find("/")
            if mainType == "text":
                cte="quoted_printable"
                quopri.encode(f,contentsEncoded,1)  #设置1将对tab编码
            else:
                cte="base64"
                base64.encode(f,contentsEncoded)
            f.close()
            subMsg = email.Message.Message()
            subMsg.add_header("Content-type",ContentType,name=fileName)
            subMsg.add_header("Content-transfer-encoding",cte)
            subMsg.set_plyload(contentsEncoded.getvalue())
            contentsEncoded.close()
            mainMsg.attach(subMsg)
        f=open(outputfile,"wb")
        g=email.Generator.Generator(f)
        g.flatten(mainMsg)
        f.close()
        return None
    if __name__=='__main__':
        main()


### 拆解一个分段MIME消息

    import email.Parser
    import os,sys
    def main():
        if len(sys.argv)  != 2:
            print "Usage: %s filename " % os.path.basename(sys.argv[0])
            sys.exit(1)
        mailFile = open(sys.argv[1],"rb")
        p=email.Parser.Parser()
        msg=p.parse(mailFile)
        mailFile.close()
        partCounter = 1
        for part in msg.walk():
            if part.get_main_type() == "multipart":
                continue
            name = part.get_param("name")
            if name == None:
                name = "part-%i" % partCounter
            partCounter += 1
            #在产品代码中，应确保此名字在你的操作系统中是有效文件名
            #否则，就应该对名字做一些修改，直到它成为有效文件名
            f=open(name,"wb")
            f.write(part.get_payload(decode=1))
            f.close()
            print name
    if __name__ == "__main__":
        main()

### 删除邮件消息中的附件

    ReplFormat = """
        This message contained an attechment that was stripped out.
        This filename was : %(filename)s,
        The original type was : %(content_type)s
        (and it had additional parameters of %(params)s)
        """
    import re
    BAD_CONTENT_RE=re.compile('application/(msword|msexcel)',re.I)
    BAD_FILEEXT_RE=re.compile(r'(\.exe|\.zip|\.pdf|\.scr|\.ps)$')
    def sanitise(msg):
        '''剥去消息中所有可能的危险载荷'''
        ct=msg.get_content_type()
        fn=msg.get_filename()
        if BAD_CONTENT_RE.search(ct) or (fn and BAD_FILEEXT_RE.search(fn)):
            #有威胁的部分，先获取用于报告的信息，然后销毁
            #将content-type,键列表、值对的参数表示成key=value
            #的形式，并以逗号分割
            params = msg.get_parames()[1:]
            params = ', '.join('='.join(p) for p in params])
            #将通知消息作为新荷载
            replace = ReplFormat % dict(contentType=ct,filename=fn,params=params)
            msg.set_payload(replace)
            #移除参数并在content-type头部设置内容
            for k,v in msg.get_params()[1:]:
                msg.del_param(k)
            msg.set_type('text/plain')
            #删除没有content-type的头部
            del msg['Content-Transfer-Encoding']
            del msg['Content-Disposition']
        else:
            #检查消息的所有子部分
            if msg.is_multipart():
                payload=[ sanitise(x) for x in msg.get_payload()]
                #用清理过的荷载列表替换
                msg.set_payload(payload)
        return msg
    if __name__ == '__main__':
        import email,sys
        m=email.message_from_file(open(sys.argv[1])
        print sanitise(m)


### 探测不活动的计算机
* HearteatClient，运行在需要监测的计算机上
    
    ```
    import socket,time
    SERVER_IP = '192.169.0.15'
    SERVER_PORT = 43278
    BEAT_PERIOD = 5
    print "Sending heartbeat to IP %s, port %d" % (SERVER_IP,SERVER_PORT)
    print "print Ctrl-C to stop"
    while True:
        hbSocket = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
        hbSocket.sendto('PyHB',(SERVER_IP,SERVER_PORT))
        if __debug__:
            print 'Time: %s' % time.ctime()
            time.sleep(BEAT_PERIOD)
    ```
* ThreadedBeatServer
    
    ```
    import socket,threading,time
    UDP_PORT = 43278
    CHECK_RERIOD = 20
    CHECK_TIMEOUT = 15
    class Heartbeats(dict):
        """用线程锁管理共享的heartbeats字典"""
        def __init__(self)：
            super(Heartbeats,self).__init__()
            self._lock=threading.Lock()
        def __setitem__(self,key,value):
            """为某个客户端创建或更新字典中的条目"""
            self._lock.acquire()
            try:
                super(Heartbeats,self).__setitem__(key,value)
            finally:
                self._lock.release()
        def getSilent(self):
            """返回沉默期长于CHECK_TIMEOUT的客户端列表
            limit=time.time()-CHECK_TIMEOUT
            self._lock.acquire()
            try:
                silent = [ip for (ip,ipTime) in self.items() if ipTime<limit]
            finally:
                self._lock.release()
            return silent
    class Receiver(threading.Thread):
        """接受UDP包并记录在heartbeats字典中"""
        def __init__(self,goOnEvent,heartbeats):
            super(Received,self).__init__()
            self.goOnEvent = goOnEvent
            self.heartbeats = heartbeats
            self.recSocket=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)
            self.recSocket.settimeout(CHECK_TIMEOUT)
            self.recSocket.bind(('',UDP_PORT))
        def run():
            while self.goOnEvent.isSet():
                try:
                    data,addr = self.recSocket.recvfrom(5)
                    if data=='PyHB':
                        self.heartbeats[addr[0]] = time.time()
                except socket.timeout:
                    pass
    def main(num_receivers=3):
        receiverEvent = threading.Event()
        receiverEvent.set()
        heartbeats=Heartbeats()
        receivers=[]
        for i in rang(num_receivers):
            receiver = Receiver(goOnEvent=receiverEvent,heartbeats=heartbeats)
            receiver.start():
            receivers.append(receiver)
        print "Threading heartbeat server listenging on port %d" % UDP_PORT
        print "press Ctrl-C to stop"
        try:
            while True:
                silent = heartbeats.getSilent()
                print "Silent clients: %s" % silent
                time.sleep(CHECK_RERIOD)
        except KeyboardInterrupt:
            print "Exiting,please wait..."
            receiverEvent.clear()
            for receiver in receivers:
                receiver.join()
            print 'Finished'
        print 'Finished.'
    if __name__=='__main__':
        main()

### 用HTTP监视网络

    import BaseHTTPServer,shutil,os
    from cStringIO import StringIO
    class MyHTTPRequestHandler(BaseHTTPServer.BaseHTTPRequestHandler):
        #我们服务的HTTP路径以及我们服务的命令行命令
        cmds={"/ping":'ping www.thinkware.se',
            "/netstat":"netstat -a",
            "/tracert":"tracert www.thinkware.se",
            "/srvstats":"net statistics server",
            "/wsstats":"net statistics workstation",
            "route":"route print"}
        def do_GET(self):
            """服务一个GET请求"""
            f=self.send_head()
            if f:
                f=StringIO()
                machine=os.popen('hostname').readlines()[0]
                if self.path == '/':
                    heading="Select a command to run on %s " % (machine)
                    body = (self.getMenu()+
                        "<p>The screen won't update utill the selected"
                        "command has finished.Please be patient.")
                else:
                    heading = "Execution of ''%s'' on %s " % (
                        self.cmds[self.path],machine)
                    cmd = self.cmds[self.path]
                    body = '<a href="/">Main Menu&lt;/a&gt;<pre>%s</pre>\n' %
                        (os.popen(cmd).read())
                f.write("<html><head><title>%s</title></head>\n" % heading
                f.write('<body><H1>%s</H1>\n' % (heading))
                f.write(body)
                f.write('</body></html>\n')
                f.seek(0)
                self.copyfile(f,self.write)
                f.close()
            return f
        def do_HEAD(self):
            """服务一个HEAD请求"""
            f=self.send_head()
            if f:
                f.close()
        def send_head(self):
            path=self.path
            if not path in ['/']+self.cmds.keys():
                head = "Command ''%s'' not found.Try one of these:<ul>" % path
                msg = head + self.getMenu()
                self.send_error(404,msg)
                return None
            self.send_response(200)
            self.send_header("Content-type","text/html")
            self.end_header()
            f=StringIO()
            f.write("A test %s \n" % self.path)
            f.seek(0)
            return f
        def getMenu(self):
            keys=self.cmds.keys()
            keys.sort()
            msg.append('<ul>'
            for k in keys:
                msg.append('<li><a hred="%s">%s ==>&lt;/a&gt;</li>' %  (k,k,self.cmds[k]))
            msg.append('</ul>
            return '\n'.join(msg)
        def copyfile(self,source,outputfile):
            shutil.copyfile(source,outputfile)
    def main(HandlerClass=MyHTTPRequestHandler,ServerClass=BaseHTTPServer.HTTPServer):
        BaseHTTPServer.test(HandlerClass,ServerClass)
    if __name__=='__main__':
        main()

### 网络端口的转发和重定向

    import sys,socket,time,threading
    LOGGING=True
    logLock=threading.Lock()
    def log(s,*a):
        if LOGGING:
            logLock.acquire()
            try:
                print '%s:%s' % (time.ctime(), (s % a))
                sys.stdout.flush()
            finally:
                logLock.release()
    class PipeThread(threading.Thread):
        pipes=[]
        pipeslock=threading.Lock()
        def __init__(self,source,sink):
            Thread.__init__(self)
            self.source=source
            self.sink=sink
            log('Creating new pip thread %s % (%s -> %s)',
                self,source.getpeername(),sink.getpeername())
            self.pipeslock.acquire()
            try:
                self.pipes.append(self)
            finally:
                self.pipeslock.release()
            self.pipeslock.acquire()
            try:
                pipes_now=len(self.pipes)
            finally:
                self.pipeslock.release()
                log('%s pipes now active' pipes_now)
        def run(self):
            whilt True:
                try:
                    data=self.source.recv(1024)
                    if not data:break
                    self.sink.send(data)
                except:
                    break
            log('%s terminating',self)
            self.pipeslock.acquire()
            try:
                self.pipes.remove(self)
            finally:
                self.pipeclock.release()
            self.pipeslock.acquire()
            try:
                pipes_left=len(self.pipes)
            finally:
                self.pipeslock.release()
            log('%s pipes still active',pipes_left)
    class Pinhole(threading.Thread):
        def __init__(self,port,newhost,newport):
            Thread.__init__(self)
            log('Redirection:localhost:%s -> %s:%s' ,port,newhost,newport)
            self.newhost=newhos
            self.newport =newport
            self.sock=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
            self.sock.bind(('',port))
            self.sock.listen(5)
        def run(self):
            while True:
                newsock,address=self.sock.accept()
                log('Creating new session for "%s:%s"'，*address)
                fwd=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                fwd.connect((self.newhost,self.newport))
                PipeThread(newsock,fwd).start()
                PipeThread(fwd,newsock).start()
    if __name__=='__main__':
        print "Starging Pinhole port forwarder/redirector"
        import sys
        try:
            port = int(sys.argv[1])
            newhost=sys.argv[2]
            try:
                newprot=int(sys.argv[3])
            except IndexError:
                newport = port
        except (ValueError,IndexError):
            print "Usage: %s port newhost [newport]"  % sys.argv[0]
            sys.exit(1)
        sys.stdout=open('piphole.log','w')
        Pinhole(port,newhost,newport).start()
    
