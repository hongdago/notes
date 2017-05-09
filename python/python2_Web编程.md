Web编程(python2)
=====================
### 测试CGI是否在工作
    
    #!/usr/bin/env python
    print "Content-type:text/html"
    print
    print "<html><head><title>Situation snapshot</title></head><body><pre>"
    import sys
    sys.stderr = sys.stdout
    import os
    from cgi import escape
    print "<strong>Python %s</strong>" % sys.version
    keys=os.environ.keys()
    keys.sort()
    for k in keys:
        print "%s:\t%s" % (escape(k),escape(os.environ[k]))
    print "</pre></body></html>"

### 用CGI脚本处理URL

    import os,string
    def isSSL():
        return os.environ.get('SSL_PROTOCOL')
    def getScriptname()：
        """返回URL("/path/to/my.cgi")的脚本名部分
        return os.environ.get('SCRIPT_NAME','')
    def getPathinfo():
        """返回URL的剩余部分"""
        pathinfo=os.environ.get('PATH_INFO','')
        #修复IIS/4.0中的一个著名的BUG
        if os.name == 'nt':
            scriptname=getScriptname()
            if pathinfo.startswith(scriptname):
                pathinfo = pathinfo[len(scriptname):]
        return pathinfo
    def getQualifiedURL(uri=None):
        """返回带有模式、服务器名、端口的URL
        指定的uri将在服务器根URL之后"""
        schema,stdport=(('http','80'),('https','443'))[isSSL()]
        host=os.environ.get('HTTP_HOST','')
        if not host:
            host = os.environ.get('SERVER_NAME','localhost')
            port=os.environ.get('SERVER_PORT','80')
            if port != stdport:
                host = host+":"+port
        result="%s://%s" % (schema,host)
        if uri:
            result=result+uri
        return result
    def getBaseURL():
        """返回个一合法的URL"""
        return getQualifiedURL(getScriptname())


### 用CGI上传文件

    #!/usr/bin/evn python
    import cgitb;cgitb.enable()
    import os,sys
    try:
        import msvcrt #在windows上？
    except ImportError:pass #不是，也没关系
    else:
        for fd in (0,1): # 是，需要叫I/O设成二进制模式
            msvcrt.setmode(fd,os.O_BINARY)
    UPLOAD_DIR="/tmp"
    HTML_TEMPLATE="""
    <html><head><title>UPLOAD Files</title>
    </head><body><h1>Upload Files</h1>
    <form action="%(SCRIPT_NAME)s" method="POST" enctype="multipart/form-data">
    File name:<input name="file_1" type="file"><br>
    File name:<input name="file_2" type="file"><br>
    File name:<input name="file_3" type="file"><br>
    <input name="submit" type="submit">
    </form></body></html>"""
    def print_html_from():
        """将表单输出到stdout，表单提交动作也设置到脚本本身"""
        print "content-type:text/html;charset=iso-8859-1\n"
        print HTML_TEMPLATE % {'SCRIPTNAME':os.environ['SCRIPT_NAME']}
    def save_uploaded_file(form_field,upload_dir):
        """将上传的文件存入磁盘，form_field是表单中文本框
        输入的文件名，如果文本框或文件缺失，不做处理"""
        form=cgi.FieldStorage()
        if not form.has_key(form_field):return
        fileitem =form[form_field]
        if not fileitem.file:return
        fout = open(os.path.join(upload_dir,fileitem.filename),'wb')
        while True:
            chunk = fileitem.file.read(100000)
            if not chunk:break
            fout.write(chunk)
        fout.close()
    save_uploaded_file("file_1",UPLOAD_DIR)
    save_uploaded_file("file_2",UPLOAD_DIR)
    save_uploaded_file("file_3",UPLOAD_DIR)
    print_html_from()


### 检查web页面的存在

    import httplib,urlparse
    def httpExists(url):
        host,path = urlparse.urlsplit(url)[1:3]
        if ':' in host:
            #指定了端口，试图使用它
            host,port = host.split(':',1)
            try:
                port=int(port)
            except ValueError:
                print "Invalid port number %r" % port
                return False
        else:
            #未指定端口，是有默认的
            port = None
        try:
            connection = httplib.HTTPConnection(host,port=port)
            connection.request('HEAD',path)
            resp = connection.getresponse()
            if resp.status == 200: #正常的"找到了"状态
                found = True
            elif resp.status == 302: #对临时重定向递归
                found = httpExists(urlparse.urljoin(url,resp.getheader('location','')))
            else:
                print "Status %d%s:$s" % (resp.status,resp.reason,url)
                found = False
        except Eception,e:
            print e.__class__,e,url
            found = False
        return found
        

### 通过HTTP检查内容类型

    
    import urllib
    def isContentType(URLorFile,contentType='text'):
        """判断URL(urllib.urlopen访问的伪文件对象）
        是否是要求的类型（默认为文本)
        """
        try:
            if isinstance(URLorFile,str):
                thefile=urllib.urlopen(URLorFile)
            else:
                thefile=URLorFile
            result = thefile.info().getmaintype() == contentType.lower()
            if thefile is not URLorFile：
                thefile.close()
        except IOError:
            result = False #如果我们无法打开它，它什么类型也不是
        return result


### 续传HTTP下载文件

    import urllib,os
    class MyURLOpener(urllib.FancyURLopener):
        """子类化以允许err 206（部分文件被传送),对我们而言这不是错误"""
        def http_error_206(self,url,fp,errcode,errmsg,headers,data=None):
            pass #忽略这个意料之中的"错误"码
    def getrest(dlFile,fromUrl,verbose=0):
        myUrlclass=MyURLOpener()
        if os.path.exists(dlFile):
            outputFile = open(dlFile,"ab")
            existSize = os.path.getsize(dlFile)
            #如果文件存在，下载剩余部分
            myUrlclass.addheader("Range",'bytes=%s-' % (existSize))
        else:
            outputFile =open(dlFile,'wb')
            existSize = 0
        webPage = myUrlclass.open(fromUrl)
        if verbose:
            for k,v in webPage.headers.items():
                print k,'=',v
        #如果我们已经有了整个文件，没必要再下载一遍
        numBytes = 0
        webSize = int(webPage.headers['Content-Length'])
        if webSize == existSize:
            if verbose:
                print "File (%s) was already download from URL (%s)" %
                    (dlFile,fromUrl)
        else:
            if verbose:
                print "Downloading %d more bytes" % (webSize-existSize)
            while True:
                data = webPage.read(8192)
                if not data:
                    break
                outputFile.wrtie(data)
                numBytes =numBytes + len(data)
        webPage.close()
        outputFile.close()
        if verbose:
            print "downloaded",numBytes,'bytes from',webPage.url
        return numbytes



### 通过带身份验证的代理进行HTTPS导航

    import httplib,base64,socket
    #脚本的参数
    user="proxy_login"
    passwd="proxy_pass"
    host="login.yahoo.com"
    port=443
    phost="proxy_host"
    pport=80
    #设置验证信息
    user_pass=base64.encodestring(user+":"+passwd)
    proxy_authorization='Proxy-authorization: Basic '+user_pass+'\r\n'
    proxy_connect="CONNECT %s:%s HTTP/1.0\r\n" % (host,port)
    user_agent='User-Agent: python\r\n'
    proxy_pieces = proxy_connect+proxy_authorization+user_agent+'\r\n'

    #链接到代理
    proxy_socket=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    proxy_socket.connect((phost,pport))
    proxy_socket.sendall(proxy_pieces+'\r\n')
    response = proxy_socket.recv(8192)
    status = response.split()[1]
    if status != '200':
        raise IOError('Connecting to proxy: status=%s' % status)
    #设置SSL socket
    ssl=socket.ssl(proxy_socket,None,None)
    sock = httplib.FakeSocket(proxy_socket,ssl)
    #初始化httplib，并用SSL Socket 来替换连接的socket
    h=httplib.HTTPConnection('localhost')
    h.sock=sock
    #最后，使用已经变成HTTPS的httplib
    h.request('GET','/')
    r=h.getresponse()
    print r.read()
    
