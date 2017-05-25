算法(python2)
=======================
### 消除序列中的重复

    def unique(s):
        """返回一个无序的列表，其中没有重复"""
        #首先使用set,因为通常这是最快的可行方法
        try:
            return list(set(s))
        except TypeError:
            pass
        """换到另一种方法，由于你无法对元素采用哈希，只好
        尝试排序，这会把相等的元素集中到一起，从而便于删除
        t = list(s)
        try:
            t.sort()
        except TypeError:
            del t #换到另一种方法
        else：
            #排序可行，删除重复项
            return [x for i,x in enumerate(t) if not i or x != t[i-1]]
        #暴力法是最后的手段
        u=[]
        for x in s:
            if x not in u:
                u.append(x)
        return u

### 在保留序列顺序的前提下消除其中的重复

    #f定义了序列seq的元素之间的等价对应关系，而且
    #对于seq的任意元素x,f(x)必须是可哈系的
    def unique(seq,f=None):
        """保留由f定义的每个等价类中最早出现的元素"""
        if f is None:
            def f(x): return x
        already_seen=set()
        result=[]
        for item in seq:
            marker =f(item):
            if marker not in already_seen:
                already_seen.add(marker)
                result.append(marker)
        return result;


### 缓存函数的返回值

    #手工缓存化函数
    fib_memo={}
    def fib(n):
        if n<2 : return 1
        if n not in fib_memo:
            fib_memo[n] =fib(n-1)+fib(n-2)
        return fib_memo[n]

    #闭包
    def memoize(fn):
        memo={}
        def memoizer(*param_tuple,**kwds_dict):
            #如果有命名参数将无法缓存化
            if kwds_dict:
                return fn(*param_tuple,**kwds_dict)
            try:
                #试图使用memo字典，若失败则更新
                try:
                    return memo[param_tuple]
                except KeyError:
                    memo[param_tuple] = result = fn(*param_tuple)
                    return result
            except TypeError
                #一些可变的参数，绕过缓存化机制
                retrun fn(*param_tuple)
        return memoizer
    
    @memoize
    def fib(n):
        if n<2:return 1
        retrun fib(n-1)+fib(n+2)

### 实现一个FIFO容器

    class Fifo(list):
        def __init__(self):
            self.back=[]
            self.append=self.back.append
        def pop(self):
            if not self:
                self.back.reverse()
                self[:] = self.back
                del self.back[:]
            return super(Fifo,self).pop()

    class FifoList(list):
        def pop(self):
            return super(FifoList,self).pop(0)

    class FifoDict(dict):
        def __init__(self):
            self.nextin=0
            self.nextout=0
        def append(self,data):
            self.nextin+=1
            self[self.nextin]=data
        def pop(self):
            self.nextout+=1
            return dict.pop(self,self.nextout)

    class FifoDeque(collections.deque):
        pop=collections.deque.popleft

### 使用FIFO策略来缓存对象

    import UserDict
    class FifoCache(object,UserDict.DictMixin):
        '''一个能够记住被设置过的条目的映射'''
        def __init_-(self,num_entries,dct=()):
            self.num_entries = num_entries
            self.dct=dict(dct)
            self.lst=[]
        def __repr__(self):
            return "%r(%r,%r)" % (
                self.__class__.__name__,self.num_entries,self.dct)
        def copy(self):
            return self.__class__(self,num_entries,self.dct)
        def keys(self):
            return list(self.lst)
        def __getitem__(self,key):
            return self.dct[key]
        def __setitem__(self,key,value):
            dct=self.dct
            lst=self.lst
            if key in dct:
                lst.remove(key)
            dct[key]=value
            lst.append(key)
            if len(lst) > self.num_entries:
                del dct[lst.pop(0)]
        def __delitem__(self,key):
            self.dct.pop(key)
            self.lst.remove(key)
        #一个纯粹出于优化而被显示定义的方法
        def __contains__(selg,item):
            return item in self.dct
        has_key=__contains__


### 将整数格式化为二进制字符串

    def _bytes_to_bits():
        """准备并返回一个前256个整数的二进制字符串的列表
        准备一个长度一致的表，填以占位符号"""
        the_table=256*[None]
        #我们会不断地循环[7,6,...,1,0],所以一次生成
        bits_per_byte =range(7,-1,-1)
        for n in xrange(256):
            #准备第n个字符：以8个占位符的列表作为开端
            bits=8*[None]
            k=n
            for i in bits_per_byte:
                #获得第i位的字符串，右移n位
                bits[i]='01'[k&1]
                k=k>>1
            #将8个"0"或"1"字符拼接成字符串放入表中
            the_table[n]=''.join(bits)
        return the_table
    _bytes_to_bits=_bytes_to_bits()

    def binary(n):
        #条件，只支持非负整数
        assert n>0
        bits=[]
        while n:
            bits.append(_bytes_to_bits()[n&255])
            n= n >> 8
        #我们需要它从左到右，反转之
        bits.reverse()
        #去除领头的0,但是确保至少还留一个
        return ''.join(bits).lstrip('0') or '0'

    def binary_slow(n):
        assert n>0
        bits=[]
        while n:
            bits.append('01'[n&1])
            n= n >>1
        bits.reverse()
        return ''.join(bits) or '0'

    def bin_with_sign(n):
        if n<0 : return '-'+binary(-n)
        else: return binary(n)

    def bin_twos_complement(n,bits_per_word=16):
        #二进制补码
        if n<0: n=2<<bits_per_word) +n
        return binary(n)

    def bin_fixed(n,bits_per_word=16):
        return bin_twos_complement(n,bits_per_word).rjust(bits_per_word,'0')

### 以任意数为基数将整数格式化为字符串

    import string
    def format(number,radix,digits=string.digits+string.ascii_lowercase):
        """使用给定的'digits'(默认为数字和小写ascii字母),以指定的基(radix),
        格式化给定的整数(number)"""
        if not 2<=radix<=len(digits):
            raise ValueError,"radix must be in 2..%r,not %r" % (len(digits),radix)
        #已自然的顺序创建一个digit的列表（最小有效位在最左端）
        #最后将其翻转，并拼接成一个字符串
        result=[]
        addon=result.append
        #计算'sign'
        if number <0:
            number=-number
            sign='-'
        elif number ==0:
            sign='0'
        _divmod=divmod   #对局部变量的访问更快
        while number:
            number,rdigit=_divmod(number,radix)
            addon(digits[rdigit])
        addon(sign)
        result.reverse()
        return ''.join(result)

    if __name__=='__main__':
        as_str="qwer"
        as_num= 1255059
        num=int(as_str,36)
        assert num == as_num
        res=format(num,36)
        assert res==as_str

    
