面向对象(python2)
================
### 温标的转换

    class Temperature(object):
        coefficients={'c':(1.0,0.0,-273.15),'f':(1.8,-273.15,32.0),
            'r':(1.8,0.0,0.0)}

            def __init__(self,**kwargs):
                #默认是绝对（开氏）温度0,但可接受一个命名的参数
                #名字可以是k,c,f或r，分别对应不同的温标
                try:
                    name,value = kwargs.popitem()
                except KeyError:
                    #无参数 默认k=0
                    name,value="k",0
                #若参数过多，或参数不能够被识别，报错
                if kwargs or name not in 'kcfr':
                    kwargs[name] = value
                    raise TypeError,'invalid arguments %r' % kwargs
                setattr(self,name,float(value))
            def __getattr__(self,name):
                #将c,f,r映射到k的计算
                try:
                    eq=self.coefficients[name]
                except KeyError:
                    #未知名字，提示错误
                    raise AttributeError,name
                return (self.k+eq[1]) * eq[0] + eq[2]
            def __setattr__(self,name,value):
                # 将对k,c,f,r的设置映射到对k的设置;并禁止其他的选项
                if name in self.coefficients:
                    #名字是c,f,r，计算并设置k
                    eq=self.coefficients[name]
                    self.k=(value-eq[2])/eq[0]-eq[1]
                elif name == 'k':
                    object.__setarrt(self,name,value)
                else:
                    raise AttributeError,name
            def __str__(self):
                return "%s K" % self.k
            def __repr__(self):
                return "Temperature(k=%r)" % self.k

### 定义常量
    
    #创建一个模块const.py
    class _const(object):
        class ConstError(TypeError):pass
        def __setattr__(self,name,value):
            if name in self.__dict__:
                raise self.ConstError,"Can't rebind const(%s)" % name
            self.__dict__[name]=value
        def __delattr__(self,name):
            if name is self.__dict__:
                raise self.ConstError,"Can't unbind const(%s)" % name
    import sys
    sys.modules[__name__]=_const()

    #使用
    const.magic=88
    del cost.magic

### 限制属性的设置

    def no_new_attributes(wrapped_setattr):
        """试图添加新属性时，报错
        但允许已存在的属性被随意设置"""
        def __setattr__(self,name,value):
            if hasattr(self,name): #非新属性，允许
                wrapped_setattr(self,name,value)
            else:
                raise AttributeError("Can't add attribute %r" % name)
        return __setattr__
    class NotNewAttrs(object):
        """NotNewAttrs的子类会拒绝新属性的添加
        但是允许已存在的属性被赋予新值"""
        #向此类的实例添加新属性的操作被屏蔽
        __setattr__=no_new_attributes(object.__setattr__)
        class __metaclass__(type):
            "一个简单的自定义元类，禁止向类添加属性"
            __setattr__=no_new_attributes(type.__setattr__)
            
### 链式字典查询

    class Chainmap(object):
        def __init__(self,*mappings):
            #记录映射的序列
            self._mappings=mappings
        def __getitem__(self,key):
            #在序列的字典中查询
            for mapping im slef._mappings:
                try:
                    return mapping[key]
                except KeyError:
                    pass
            #没有在任何字典中找到key,所以抛出KeyError
            raise KeyError,key
        def get(self,key,default=None):
            #若self[key]存在则返回之，否则返回default
            try:
                return self[key]
            except KeyError:
                return default
        def __contains__(self,key):
            try:
                self[key]
                return True
            except KeyError:
                return False

    #使用示例
    import Chainmap
    pylookup=Chainmap(locals(),globals(),vars(__builtin__))

### 继承的替代方案---自动托管
* 需要从某个类或者类型继承，但是需要对继承做一些调整，比如，需要选择性地隐藏某些基类的方法，而继承并不能做到这一点

    ```
    try：
        set
    except NameError：
        from sets import Set as set
    class ROError(AttributeError):pass
    class ReadOnly:
        mutators={
            list:set('''__delitem__ __delslice__ __iadd__ __imul__
                        __setitem__ __setslice__ append extend insert
                        pop remove sort'''.split())
            dict:set('''__delitem__ __setitem__ clear pop popitem
                        setdefault update'''.split())
            }
        def __init__(self,o):
            object.__setattr__(self,'_o',o)
            object.__setattr__(self,'_no',self.mutators.get(type(o),()))
        def __setattr__(self,n.v):
            raise ROError("Can't set attr %r on RO object" % n)
        def __delattr__(self,n):
            raise ROError("Can't del attr %r from RO object" % n)
        def __getattr__(self,n)
            if n in self._no:
                raise ROError("Can't get attr %r from RO object" % n)
            retun getattr(self._o,n)
    ```

### 缓存环的实现

    class RingBuffer(object):
        """这是一个未填满的缓存类"""
        def __init__(self,size_max):
            self.max=size_max
            self.data=[]
        class __Full(object):
            """这是一个已填满的缓存类"""
            def append(self,x):
                """加入新的元素覆盖最旧的元素"""
                self.data[self.cur]=x
                self.cur=(self.cur +1) % self.max
            def tolist(self):
                """已正确的顺序返回元素列表"""
                return self.data[self.cur:]+self.data[:self.cur]
        def append(self,x):
            """在缓存的末尾增加一个元素"""
            self.data.append(x):
            if len(self.data) == self.max:
                self.cur=0
                #永久性的将self的类从非满修改为满
                self.__class__=self.__Full
        def tolist(self0：
            """返回一个从最旧到最新的元素的列表"""
            return self.data

    #另一种实现
    class RingBuffer(object):
        def __init__(self,max_size):
            self.max=max_size
            self.data=[]
        def _full_append(self,x)
            self.data[self.cur] = x
            self.cur=(self.cur+1) % self.max
        def _full_get(self):
            return self.data[self.cur:]+self.data[:cur]
        def append(self,x):
            self.data.append(x)
            if len(self.data) == self.max:
                self.cur=0
                #将self.methods 从非满切换到满
                self.append=_full_append
                self.tolist=_full_get
        def tolist(self0：
            return self.data
            

### 实现状态设计模式

    class TraceNormal(object):
        '正常的状态'
        def startMessage(self):
            self.nstr=self.characters = 0
        def emitString(self,s):
            self.nstr +=1
            self.characters += len(s)
        def endMessage(self):
            print '%d characters in %d strings' % (self.characters,self.nstr)
    class TraceChatty(object):
        '详细的状态'
        def startMessage(self):
            self.msg=[]
        def emitString(self,s):
            self.msg.append(str(s))
        def endMessage(self):
            print 'Message: ',', '.join(self.msg)
    class TraceQuiet(object);
        '无输出状态'
        def startMessage(self):pass
        def emitString(self,s):pass
        def endMessage(self):pass
    class Tracer(object):
        def __init__(self,state):self.state=state
        def setState(self,state):self.state=state
        def emitString(self,strings):
            self.state.startMessage()
            for s in strings:
                self.state.emitString(s)
            self.state.endMessage()
    if __name__ == '__main__':
        t=Tracer(TraceNormal())
        t.emitString('some example strings here'.split())
        t.setState(TraceQuiet())
        t.emitString('some example strings here'.split())
        t.setState(TraceChatty())
        t.emitString('some example strings here'.split())

### 用__init__参数自动初始化实例变量

    def attributesFromDict(d):
        self=d.pop('self')
        for n,v in d.iteritem():
            setattr(self,n,v)
    def __init__(self,foo,bar,baz,boom=1,bang=2):
        attributesFromDict(locals())

    def attributesFromArguments(d):
        self=d.pop('self')
        codeObject=self.__init__.im_func.func_code
        argumentNames=codeObject.co_varnames[1:codeObject.co_argcount]
        for n in argumentNames:
            setattr(self,n,d[n])

### 调用超类的__init__方法

    class NewStyleOnly(A,B,C):
        def __init__(self):
            super(NewStyleOnly,self).__init__()

    class LookBeforeYouLeap(X,Y,Z):
        def __init__(self):
            for base in self.__clas__.__bases__:
                if hasattr(base,'__init__')
                    base.__init__(self)

