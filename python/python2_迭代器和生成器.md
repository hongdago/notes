迭代器和生成器.md(python2)
===============================
### 生成Fibonacci序列

    def fib():
        '''Fibonacci数的无界生成器'''
        x,y=0,1
        while True:
            yield x
            x,y=y,x+y
    if __name__=='__main__':
        import itertools
        print list(itertools.islice(fib(),10))

### 在多重赋值中拆解部分项

    def peel(iterable,arg_cnt=1):
        """获得一个可迭代对象的前arg_cnt项，
        然后用一个迭代器表示余下的部分"""
        iterator=iter(iterable)
        for num in xrange(arg_cnt):
            yield iterator.nect()
        yield iterator
    if __name__=='__main__':
        t5=range(1,6)
        a,b,c=peel(t5,2)
        print a,b,list(c)
        #输出:1,2,[3,4,5]

### 自动拆解出需要的数目的项

    import inspect,opcode
    def how_many_unpacked():
        f = inspect.currentframe().f_back.f_back
        if(ordf.f_code,co_code[f.f_lasti])==opcode.opmap['UNPACK_SEQUENCE']:
            return ord(f.f_code.co_code[f.f_lasti+1])
        raise ValueError("Must be a generator on RHS of a mutiple assignment!")
    def unpack(iterable):
        iterator=iter(iterable)
        for num in xragne(how_many_unpacked()-1):
            yield iterator.next()
        yield iterator
    if __name__=='__main__':
        t5=range(1,6)
        a,b,c=unpack(t5)
        print a,b, list(c)

### 逐段读取文本文件

    def paragraphs(lines,is_separator=str.isspace,joiner=''.join):
        paragraph=[]
        for line in lines:
            if is_separator(line):
                if paragraph:
                    yield joiner(paragraph)
                    paragraph=[]
            else:
                paragraph.append(line)
        if paragraph:
            yield joiner(paragraph)


### 读取带有延续符的行

    def logical_lines(physical_lines,joiner=''.join):
        logical_line=[]
        for line in physical_lines:
            stripped=line.rstrip()
            if stripped.endswith('\\'):
                logical_line.append(stripped[:-1]
            else:
                """没有延续行的一行,逻辑行的结束"""
                logical_line.append(line)
                yield joiner(logical_line)
                logical_line=[]
        if logical_line:
            yield joiner(logical_line)


### 将一个数据快流处理成行流

    def ilines(source_iterable,eol='\r\n',out_eol='\n'):
        tail = ''
        for block in source_iterable:
            pieces = (tail+block).split(eol)
            tail=pieces.pop()
            for line in pieces:
                yield line+out_eol
        if tail:
            yield tail

