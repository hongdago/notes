持久化和数据库(python2)
==========================
### 使用marshal模块序列化对象
    
    data={12:"twelve",'feep':list('ciao'),1.23:4+5j,(1,2,3):u'wer'}
    import marshal
    bytes=marshal.dumps(data)

    redata=marshal.loads(bytes)

    ouf=open('datafile.dat','wb')
    marshal.dump(data,ouf)
    marshal.dump('some string',ouf)
    marshal.dump(range(10),ouf)
    ouf.close()

    inf=open('datafile.dat','rb')
    redata=marshal.load(inf)
    restr=marshal.load(inf)
    reli=marshal.load(inf)
    inf.close()

### 使用pickle和cPickle模块序列化数据

    
    data={12:"twelve",'feep':list('ciao'),1.23:4+5j,(1,2,3):u'wer'}
    import cPickle
    text=cPickle.dumps(data)
    bytes=cPickle.dumps(data,2)

    redata=cPickle.loads(text)
    redata=cPickle.loads(bytes)

    ouf=open('datafile.dat','w')
    cPickle.dump(data,ouf)
    cPickle.dump('some string',ouf)
    cPickle.dump(range(10),ouf)
    ouf.close()

    inf=open('datafile.dat','r')
    redata=marshal.load(inf)
    restr=marshal.load(inf)
    reli=marshal.load(inf)
    inf.close()


### 在Pickling的时候压缩

    import cPickle,gzip
    def save(filename,*objects):
        '''将对象存为压缩过的磁盘文件'''
        fil=gzip.open(filename,'wb')
        for obj in objects:
            cPickle.dump(obj,fil,protocol=2)
        fil.close()
    def load(filename):
        fil=gzip.open(filename,'rb')
        while True:
            try:
                yield cPickle.load(fil)
            except EOFError:
                break
        fil.close()


### 对类和实例使用cPickle模块

    """处理文件对象不能被序列化的问题"""
    class PrettyClever(object):
        def __init__(self,*stuff):
            self.stuff=stuff
        def __getstate__(self):
            def normalize(x):
                if isinstance(x,file):
                    return 1,(x.name,x.mode,x.tell())
                return 0,x
            return [normalize(x) for x in self.stuff]
        def __setstate__(self,stuff):
            def reconstruct(x):
                if x[0] == 0:
                    return x[1]
                name,mode,offs=x[1]
                openfile=open(name,mode)
                openfile.seek(offs)
                return openfile
            self.stuff=tuple([reconstruct(x) for x in self.stuff])

### Pickling被绑定的方法

    class picklable_boundmethod(object):
        def __init__(self,mt):
            self.mt=mt
        def __getstate__(self):
            return self.mt.im_self,self.mt.im_func.__name__
        def __setstate__(self,(s,fn)):
            self.mt=getattr(s,fn)
        def __call__(self,*a,**kw):
            return self.mt(*a,**kw)

### 通过shelve修改对象

    >>>import shelve
    >>>she=shelve.open('try.she','c')
    >>>for c in "spam":she[c]={c:23}
    ...
    >>>for c in she.keys():print c,she[c]
    ...
    a {'a': 23}
    s {'s': 23}
    m {'m': 23}
    p {'p': 23}
    >>>she.close()

    >>>she=shelve.open('try.she','c')
    >>>she['p']
    {'p': 23}
    >>>she['p']['p']=42   #临时对象，不会回写
    >>>she['p']
    {'p':23}
    
    >>>a=she['p']
    >>>a['p']['p']=42
    >>>she['p']=a         #触发回写
    >>>she['p']
    {'p':42}

### 访问MySQL

    import pymysql
    #创建一个连接对象
    conn=pymysql.connect(host='127.0.0.1',port=3306,user='joe',passwd='secret',db='test',charset='utf8')
    cursor=conn.cursor()
    #执行一个sql
    sql="select * from Users"
    cursor.execute(sql)
    #从游标中取出所有记录并关闭链接
    result=cursor.fetchall()
    conn.close()

### 在MySQL数据库中存储BLOB

    import pymysql,cPickle
    #连接到数据库，并获得游标
    conn=pymysql.connect(host='127.0.0.1',port=3306,user='joe',passwd='secret',db='test',charset='utf8')
    cursor=conn.cursor()
    #创建一个新表用于实验
    cursor.execute('CREATE TABLE justatest (name TEXT,ablob BLOB)')
    try:
        #准备一些BLOB用于测试
        names="aramis","athos","porthos"
        data={}
        for name in names:
            datum=list(name)
            datum.sort()
            data[name]=cPickle.dumps(datum,2)
        sql="INSERT INTO justatest values (%s,%s)"
        for name in names:
            cursor.execute(sql,(name,pymysql.escape_string(data[name])))
        #恢复数据检查
        sql="select name,ablob from justatest order by name"
        cursor.execute(sql)
        for name,blob in cursor.fetchall():
            print name,cPickle.loads(blob),cPickle.loads(data[name])
    finally:
        cursor.execute('DROP TABLE justatest')
        conn.close()

### 在SQLite中存储BLOB
    
    import sqlite,cPickle
    class Blob(object):
        '''自动转换二进制串'''
        def __init__(self,s):
            self.s=s
        def _quote(self):
            return "'%s'" % sqlite.encode(self.s)
    #创建一个测试的内存数据库,获得游标，创建一个表
    connection=sqlite.connect(":memory:")
    cursor=connection.cursor()
    cursor.execute('CREATE TABLE justatest (name TEXT,ablob BLOB)')'
    #准备一些BLOB用于测试
    names="aramis","athos","porthos"
    data={}
    for name in names:
        datum=list(name)
        datum.sort()
        data[name]=cPickle.dumps(datum,2)
    sql="INSERT INTO justatest values (%s,%s)"
    for name in names:
        cursor.execute(sql,(name,Blob(data[name]))
    #恢复数据检查
    sql="select name,ablob from justatest order by name"
    cursor.execute(sql)
    for name,blob in cursor.fetchall():
        print name,cPickle.loads(blob),cPickle.loads(data[name])
    conn.close()

### 生成一个字典将字段名映射为列号

    def fields(cursor):
        """假设游标对象已经执行并返回"""
        results={}
        for column,desc in enumerate(cursor.description):
            results[desc[0]]=column
        return results
