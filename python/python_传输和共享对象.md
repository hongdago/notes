python_传输和共享对象.md
=============================
> 当我们序列化一个对象时，通常会做某种**表述性状态传输(Representational State Transfer REST)**。当我们序列化对象时，实际上是在创建对象状态的表示，这种表示可以被传输到另外一个进程(通常在另外一个主机上),另外一个进程可以根据这个状态的表示和一个本地定义的类来创建原始对象的对应版本。

### 用REST实现CRUD操作
一个REST服务器通常通过下面5中基本用法支持CRUD操作
* 创建:我们会用HTTP的POST请求创建一个新的对象，而URI在这种情况下只提供类信息。一个类似以//HOST/app/blog/的路径可能可以为类命名。响应可以是201,并且包含最后被保存对象的备份。返回的对象可能包括RESTful服务器为新创建对象分配的URI或者用于创建URI的相关键。POST请求应该被用于创建一些新的RESTful资源
* 获取-搜索(Retrieve-Search):用于获取多个对象的请求。我们会使用HTTP的GET请求，并且包含一个提供查询条件的URI，通常条件以查询字符串的形式包含在？字符之后。可能的URI是//HOST/app/blog/?title="2013-2014"。注意，GET不会改变任何RESTful资源的状态
* 获取-实例(Retrieve-Instance):这是获取单个对象的请求，我们会使用GET请求，并且包含一个在路径中指定了特殊对象的URI。可能的URI是//HOST/app/blog/id/
。尽管预期的响应是一个单独的对象，但是为了与搜索的响应兼容，它有可能被保存在一个列表中。
* 更新:我们会用HTTP的PUT请求，并且包含一个指定了目标替代对象的URI。可能的URI是//HOST/app/blog/id/。相应可以是200,并且包含一份更新后对象的备份。
* 删除:我们会用HTTP的DELETE请求，并且包含一个类型//HOST/app/blog/id/的URL。响应可以是简单的204 NOT CONTENT,而不用在响应中提供任何对象的细节。

### 创建简单的REST应用程序和服务器
>示例

    import random
    import sys
    import wsgiref.util
    import json
    import http.client
    from wsgiref.simple_server import make_server
    class Wheel(object):
        """Abstract,zero bins omitted."""
        def __init__(self):
            self.rng=random.Random()
            self.bins=[
                {str(n):(35,1),
                self.redblack(n):(1,1),
                self.hilo(n):(1,1),
                self.evenodd(n):(1,1),
                } for n in range(1,37)
            ]
        @staticmethod
        def redblack(n):
            return "Red" if n in (1,3,5,7,9,12,14,16,18,19,21,23,
                25,27,30,32,34,36) else "Black"
        @staticmethod
        def hilo(n):
            return "Hi" if n >= 19 else "Lo"
        @staticmethod
        def evenodd(n):
            return "Even" if n % 2 == 0 else "Odd"
        def spin(self):
            return self.rng.choice(self.bins)

    class Zero(object):
        def __init__(self):
            super(Zero,self).__init__()
            self.bins+=[{'0':(35,1)}]

    class DoubleZero(object):
        def __init__(self):
            super(DoubleZero,self).__init__()
            self.bins +=[{'00':(35,1)}]

    class American(Zero,DoubleZero,Wheel):
        pass

    class European(Zero,Wheel):
        pass

    american=American()
    european=European()

    """Wheel实例的WSGI程序"""
    def wheel(environ,start_respone):
        request=wsgiref.util.shift_path_info(environ)  #1.parse
        print("wheel",request,file=sys.stderr)  #2.loadding
        if request.lower().startswith('eu'):  #3 evaluate
            winner=european.spin()
        else:
            winner=american.spin()
        status='200 ok'
        headers=[('Content-type','application/json;character=utf-8')]
        start_respone(status,headers)
        return [json.dumps(winner).encode('utf-8')]

    """服务端"""
    def roulette_server(count=1):
        httpd=make_server('',8000,wheel)
        if count is None:
            httpd.serve_forever()
        else:
            for c in range(count):
                httpd.handle_request()

                
    """客户端"""
    def json_get(path="/"):
        rest=http.client.HTTPConnection('localhost',8000)
        rest.request("GET",path)
        response=rest.getresponse()
        print(response.status,response.reason)
        print(response.getheaders())
        raw=response.read().decode('utf-8')
        if response.status == 200:
            document=json.loads(raw)
            print(document)
        else:
            print(raw)


    if __name__ == "__main__":
        """演示RESTful服务并创建单元测试"""
        import concurrent.futures
        import time
        with concurrent.futures.ProcessPoolExecutor() as executor:
            executor.submit(roulette_server,4)
            time.sleep(2) #Wait for the server to start
            json_get()
            json_get()
            json_get("/european")
            json_get("/european")

### 使用可回调类创建WSGI应用程序

    from collections.abc import Callable
    class Wheel2(Wheel,Callable):
        def __call__(self,environ,start_response):
            winner = self.spin()  #3、evaluate
            status='200 ok'
            headers=[('Content-type','application/json;charset=utf-8')]
            start_response(status,headers)
            return [json.dumps(winner).encode('UTF-8')]

    class American2(Zero,DobleZero,Wheel2):
        pass

    class European2(Zero,Wheel2):
        pass

    class Wheel3(Callable):
        def __init__(self):
            self.am=American2()
            self.eu=European2()
        def __call__(self,environ,start_respone):
            request=wsgiref.util.shift_path_info(environ) #1.Parse
            print("Wheel3",request,file=sys.stderr) #2.Logging
            if request.lower().startswith('eu'):
                response=self.eu(environ,start_respone)
            else:
                response=self.am(environ,start_response)
            return response

### 多层REST服务
    
    from collections import defaultdict
    class Table:
        def __init__(self,stake=100):
            self.bets=defaultdict(int)
            self.stake=stake
        def place_bet(self,name,amount):
            self.bets[name] +=amount
        def clear_bets(self,name):
            self.bets=defaultdict(int)
        def resolve(self,spin):
            """spin is a dict with bet:(x:y)."""
            details=[]
            while self.bets:
                bet,amount=self.bets.popitem()
                if bet in spin:
                    x,y=spin[bet]
                    self.stake +=amount*x/y
                    details.append((bet,amount,'win'))
                else:
                    self.stake -=amount
                    details.append((bet,amount,'lose'))
            return details

    class WSGI(Callable):
        def __call__(self,environ,start_response):
            raise NotImplementedError
    class RESTException(Exception):
        pass

    class Roulette(WSGI):
        def __init__(self,wheel):
            self.table=Table(100)
            self.rounds = 0
            self.wheel=wheel
        def __call__(self,environ,start_respone):
            app=wsgiref.util.shift_path_info(environ)
            try:
                if app.lower() =="player":
                    return self.player_app(environ,start_respone)
                elif app.lower() == "bet":
                    return self.bet_app(environ,start_respone)
                elif app.lower() == "wheel":
                    return self.wheel_app(environ,start_respone)
                else:
                    raise RESTException("404 NOT_FOUND","Unknown app in {SCRIPT_NAME}/{PATH_INFO}".format(environ))
            except RESTException as e:
                status = e.args[0]
                headers=[("Content-type",'text/plain;charset=utf-8')]
                start_response(status,headers,sys.exc_info())
                return [repr(e.args).encode('utf-8')]

        def player_app(self,environ,start_response):
            if environ['REQUEST_METHOD'] == 'GET':
                details = dict(stake=self.table.stake,rounds=self.rounds)
                status='200 OK'
                headers=[('Content-type','application/json;charset=utf-8')]
                start_response(status,headers)
                return [json.dumps(details).encode('UTF-8')]
            else:
                raise RESTException('405 METHOD_NOT_ALLOWED',"Method '{REQUEST_METHOD}' not allowed".format_map(environ))

        def bet_app(self,environ,start_response):
            if environ['REQUEST_METHOD'] == 'GET':
                detail = dict(self.table.bets)
            elif environ['REQUEST_METHOD'] =='POST':
                size=int(environ['CONTENT_LENGTH'])
                raw=environ['wsgi.input'].read(size).decode("UTF-8")
                try:
                    data=json.loads(raw):
                    if isinstance(data,dict):data=[data]
                    for detail in data:
                        self.table.place_bet(detail['bet'],int(detail['amount']))
                except Exception as e:
                    raise RESTException("403 FORBIDDEN","Bet {raw!r}".format(raw=raw))
                detail = dcit(self.table.bets)
            else:
                raise RESTException('405 METHOD_NOT_ALLOWED',"Method '{REQUEST_METHOD}' not allowed".format_map(environ))
            status = '200 ok'
            headers=[('Content-type','application/json;charset=utf-8')]
            start_response(status,headers)
            return [json.dumps(details).encode('UTF-8')]

        def wheel_app(self,environ,start_response):
            if environ['REQUEST_METHOD'] == 'POST':
                size=environ['CONTENT_LENGTH']
                if size !='':
                    raw=environ['wsgi.input'].read(int(size))
                    raise RESTException("403 FORBIDDEN","Date '{raw!r}' not allowed".format(raw=raw))
                spin=self.wheel.spin()
                payout=self.table.resolve(spin):
                self.rounds +=1
                details=dict(spin=spin,payout=payout,stake=self.table.stake,rounds=self.rounds)
                status = '200 ok'
                headers=[('Content-type','application/json;charset=utf-8')]
                start_response(status,headers)
                return [json.dumps(details).encode('UTF-8')]
            else:
                raise RESTException('405 METHOD_NOT_ALLOWED',"Method '{REQUEST_METHOD}' not allowed".format_map(environ))

    #创建roulette服务器
    def roulette_server_3(count=1):
        from wsgiref.simple_server import make_server
        from wsgiref.validate import validator
        wheel=American()
        roulette=Roulette(wheel)
        debug=validator(roulette)
        httpd=make_server('',8000,debug)
        if count is None:
            httpd.serve_forever
        else:
            for c in range(count):
                httpd.handle_request()

    #创建客户端
    def roulette_client(method="GET",path="/",data=None):
        rest=http.client.HTTPConnection('localhost',8000)
        if data:
            headers=[('Content-type','application/json;charset=utf-8')]
            params=json.dumps(data).encode('UTF-8')
            rest.request(method,path,params,header)
        else:
            rest.request(method,path)
        response=rest.getresponse()
        raw=response.read().decode("utf-8")
        if 200<=response.status < 300:
            docuement=json.loads(raw)
            return document
        else:
            print(response.status,response.reason)
            print(response.getheaders())
            print(raw)

    if __name__=='__main__':
        with concurrent.futures.ProcessPoolExecutor() as executor:
            executor.submit(roulette_server_3,4)
            time.sleep(3)
            print(roulette_server("GET","/player/"))
            print(roulette_server("POST","/bet/",{'bet':'Black','amount':2}))
            print(roulette_server("GET","/bet"))
            print(roulette_server("POST","/wheel"))


### 创建安全的REST服务

    from hashlib import sha256
    import os
    class Authentication:
        iterations=1000
        def __init__(self,username,password):
            """works with bytes,Not Unicode string"""
            self.username=username
            self.salt=os.urandom(24)
            self.hash=self._iter_hash(self.iterations,self.salt,username,password)
        @staticmethod
        def _iter_hash(iterations,salt,username,password):
            seed=salt+b":"+username+b":"+password
            for i in range(iterations):
                seed=sha256(seed).digest()
            return seed
        def __eq__(self,other):
            return self.username == other.username and self.hash == other.hash
        def __hash__(self):
            return hash(self.hash)
        def __repr__(self):
            salt_x="".join("{0:x}".format(b) for b in self.salt)
            hash_x="".join("{0:x}".format(b) for b in self.hash)
            return "{username} {iterations:d} {salt} {hash}".format(
                username=self.username,iterations=self.iterations,salt=salt_x,hash=hash_x)
        def match(self,password):
            test=self._iter_hash(self.iterations,self.salt,self.username,password)
            return self.hash==test

    class Users(dict):
        def __init__(self,*args,**kw):
            super(Users,self).__init__(*args,**kw)
            self[""]=Authentication(b"__dummy__",b"Doesn't Matter")
        def add(self,authentication):
            if authentication.username == "":
                raise KeyError("Invalid Authentication")
            self[authentication.username]=authentication
        def match(self,username,password):
            if username in self and username != "":
                return self[username].match(password)
            else:
                raise self[""].match(b"Something which doesn't match")

    if __name__=='__main__':
        users=Users()
        user=Authentication(b"Aladdin",b"open sesame")
        print(user)
        users.add(user)

### WSGI验证程序

    import base64
    class Authenticate(WSGI):
        def __init__(self,users,target_app):
            self.users=users
            self.target_app=target_app
        def __call__(self,environ,start_response):
            if "HTTP_AUTHORIZATION" in environ:
                scheme,credentials = environ['HTTP_AUTHORIZATION'].split()
                if scheme == 'Basic':
                    username,password=base64.b64decode(credentials).split(b":")
                if self.users.match(username,password):
                    environ['Authenticate.username']=username
                    return self.target_app(environ,start_respone)
                status="401 UNAUTHORIZED'
                headers=[('Content-type','application/json;charset=utf-8'),
                    ("WWW-Authenticate','Basic realm="administrator@localhost"')]
                start_respone(status,heasers
                return ['Not authorized'.encode('utf-8')]
