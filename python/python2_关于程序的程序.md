关于程序的程序(python2)
=======================
### 词法分析
    
    >>> x = "abc :def:ghi      : klm\n"
    >>> fields=x.split(':')
    >>> print fields
    ['abc ', 'def', 'ghi      ', ' klm\n']
    >>> print [f.strip() for f in x.split(':')]
    ['abc', 'def', 'ghi', 'klm']

    nodes={}
    def getnode(name):
        "返回拥有指定名字的节点，必要的时候创建之”
        if name in nodes:
            node = nodes[name]
        else:
            node =nodes[name]=node(name)
        return node
    class node(object):
        """一个节点拥有名字以及从它出发的边"""
        def __init__(self,name):
            self.name=name
            self.edgelist=[]
    class edge(object):
        ""两节点之间的边"""
        def __init__(self,name1,name2):
            self.nodes=getnode(name1),getnode(name2)
            for n i self.nodes:
                n.edgelist.append(self)
        def __repr__(self):
            return self.node[0].name + self.node[1].name


### 验证字符串是否代表着一个合法的数字

    def is_a_number(s):
        try:
            float(s)
        except Exception:
            return False
        else:
            return True

    import re
    num_re=re.compile(r'^[-+]?([0-9]+\.?[0-9]*|\.[0-9]+')([eE][-+]?[0-9]+)?$
    def is_a_number(s):
        reutn bool(num_re.match(s))

### 导入一个动态生成的模块

    
    import new
    def importCode(code,name,add_to_sys_module=False):
        """code 可以是任何包含代码的对象：字符串，文件对象或
        编译过的代码对象。通过动态导入指定的代码，初始化并返回
        一个新的模块对象，而且可选地将其按照指定名字加入到sys.modules中"""
        module = new.module(name)
        if add_to_sys_module:
            import sys
            sys.modules[name]=module
        exec code in module.__dict__
        return module

### 导入一个名字在运行时被确定的模块

    def importName(modulename,name):
        """在函数环境中从一个模块导入一个命名对象"""
        try:
            module = __import__(modulename,globals(),locals(),[name])
        except ImportError:
            return None
        return getattr(module,name)

### 将参数和函数联系起来

    def curry(f,*a,**kw):
        def curried(*more_a,**more_kw):
            return f(*(a+more_a),**(kw+more_kw))
        return curried


### 在UNIX中将主脚本和python模块绑成一个可执行的文件

    #!/bin/bash
    PYTHON=$(which python 2>/dev/null)
    if [  ! -x "$PYTHON" ];then
        echo "python executable not found - cannot continue!"
        exit 1
    fi
    exec $PYTHON - $0 $@ << END_OF_PYTHON_CODE
    import sys
    version = sys.version_info[:2]
    if version < (2,3):
        print 'Sorry,need python 2.3 or better;%s.%s is too old!' % version
    sys.path.insert(0,sys.argv[1])
    def sys.argv[0:2]
    import main
    main.main()
    END_OF_PYTHON_CODE
