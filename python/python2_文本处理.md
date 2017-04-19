文本处理(python 2)
========
### 每次处理一个字符
1. 调用内建的list函数
    
    ```thelist = list(thestring)```

2. 直接使用for语句对该字符串循环遍历
    
    ```
    for c in thestring:
            do_something_with(c)
    ```

3. 列表推导
    
    ```
    results = [do_something_with(c) for c in thestring]
    ```
4. map函数
    
    ```
    resutls = map(do_something_with, the_string)
    ```

### 字符和字符值之间的转换
    
    >>>print ord('a')
    97
    >>>print chr(97)
    a
    >>>print ord(u'\u2020')
    8224
    >>>print repr(unichr(8224)
    u'\u2020'

>unicode  encode--> bytes

>bytes    decode--> unicode

### 测试一个对象是否是类字符串
    
    def isAString(anobj):
        """str和unicode都返回True"""
        reutrn isinstance(anobj,basestring)

### 字符串对齐

    >>> print '|','hej'.ljust(20),'|','hej'.rjust(20),'|','hej'.center(20),'|'
    | hej                  |                  hej |         hej          |
    >>> print 'hej'.center(20,'+')
    ++++++++hej+++++++++

### 将字符串逐字符或逐词反转
1. 逐字反转
    
    ```
    revchars=astring[::-1]
    ```
2. 逐词反转

    ```
    #不保留之前的空格
    revwords=astring.split()
    revwords.reverse()
    revwords=' '.join(revwords)             #使用空格进行join

    revwords=' '.join(astring.split()[::-1]) #一行解决

    #保留之前的空格
    import re
    revwords = re.split(r'(\s+)',astring)   #注意模式中的()
    revwords.reverse()
    revwords = ''.join(revwords)            #注意是空字符串进行join
    ```

### 检查字符串中是否包含某字符集合中的字符

    def containAny(seq,aset):
        """检查序列seq是否含有aset中的项"""
        for c in seq:
            if c in aset:
                return True
        return False

    def containAll(seq,aset):
        """检查序列seq是否包含aset中的所有项"""
        return not set(aset).difference(seq)


    #针对与字符串astr和strset的处理
    import string
    notrans=string.maketrans('','')
    def containAny(astr,strset):
        return len(strset) != len(strset.translate(notrans,astr))
    def containAll(astr,strset):
        return not strset.translate(notrans,astr)

### 简化字符串translate方法的使用

    import string
    def translator(frm='',to='',delete='',keep=None):
        if len(to) == 1:
            to = to * len(frm)
        trans = string.maketrans(frm,to)
        if keep is not None:
            allchars=string.maketrans(frm,to)
            delete = allchars.translate(allchars,keep.translate(allchars,delete))
        def translate(s):
            return s.translate(trans,delete)
        return translate

### 检查一个字符串是文本还是二进制

    from __future__ import division   #确保/不会截断
    import string
    text_characters="".join(map(chr,range(32,127)))+"\n\r\t\b"
    __null_trans=string.maketrans("","")
    def is_text(s,text_characters=text_characters,threshold=0.30):
        #若s包含空值，它不是文本
        if "\0" in s:
            return False
        #一个“空”字符串是文本
        if not s:
            return True
        #获得s的非文本字符串构成的子串
        t=s.translate(__null_trans,text_characters)
        #如果不超过30%的字符是非文本字符，s是字符串
        return len(t)/len(s) <=threshold


