搜索和排序(Python2)
=======================
### 对字典排序

    def sorteddictValues(adict):
        """先将键排序，然后由此选出对应值"""
        keys=adict.keys()
        keys.sort()
        return [adict[key] for key in keys]

### 不区分大小写对支付串列表进行排序

    def case_insensitive_sort(string_list):
        #decorate-sort-undecorate(DSU)
        auxiliary_list=[(x.lower(),x)  for x in string_list]
        auxiliary_list.sort()
        return [x[1] for x in auxiliary_list]
    
    #原生支持
    def case_insensitive_sort(string_list):
        return sorted(string_list,key=str.lower)

    #可选方案
    def case_insensitive_sort(string_list):
        def compare(a,b):return cmp(a.lower(),b.lower())
        return string_list.sort(compare)

### 根据对象额属性将对象列表排序

    def sort_by_attr(seq,attr):
        intermed = [(getattr(x,attr),i,x for i,x in enumerate(seq)]
        intermed.sort()
        return [x[-1] for x in intermed]
    def sort_by_attr_inplace(lst,attr):
        lst[:]=sort_by_attr(lsr,attr)

### 根据内嵌的数字将字符串排序

    import re
    re_digits=re.compile(r'(\d+)')
    def embedded_number(s):
        pieces=re_digits.split(s) #切成数字和非数字
        pieces[1::2]=map(int,pieces[1::2]) #将数字部分转成整数
        return pieces
    def sort_strings_with_embedded_number(alist):
        aux=[(embedded_number(s),s) for s in alist]
        aux.sort()
        return [s for __,s in aux]

### 以随机顺序处理列表的元素

    import random
    def process_all_in_random_order(data,process):
        #如果需要保证输入列表不变，或者输入列表可能是其他可迭代对象而不是列表
        #data=list(data)
        #将列表处于随机顺序
        random.shuffle(data)
        #顺序访问
        for elem in data: process(elem)

### 获取序列中最小的几个元素

    import heapq
    def isorted(data):
        data=list(data)
        heapq.heapify(data)
        while data:
            yield heapq.heappop(data)

    #获取前n个最小的元素
    import heapq
    def smallest(n,data):
        return heapq.nsmallest(n,data)

### 在排序完毕的序列中寻找元素

    import  bisect
    x_insert_point=bisect.bisect_right(L,x)
    x_is_present= L[x_insert_point-1:x_insert_point == [x]

### 选取序列中最小的第n个元素

    import random
    def select(data,n):
        """寻找第n个元素（最小的元素是第0个)"""
        #创建一个新列表，处理小于0的索引，检查索引的有效性
        data=list(data)
        if n <0 :
            n=n+len(data)
        if not 0<= n < len(data):
            raise ValueError,"Can't get rank %d out of %d " % (n,len(data))
        #主循环，看上去类似于快速排序但不需要递归
        while True:
            pivot=random.choice(data)
            pcount=0
            under,over=[],[]
            uappend,oappend=under.append,over.append
            for elem in data:
                if elem < pivot:
                    uappend(elem)
                elif elem>pivot:
                    oappend(elem)
                elst:
                    pcount +=1
            numunder=len(under)
            if n<numunder:
                data=under
            elif n<numunder + pcount:
                return pivot
            else:
                data=over
                n-=(numunder+pcount)

### 根据姓的首字母将人名排序和分组

    import itertools
    def groupnames(name_iterable):
        #name_iterable格式:名-中名-姓，
        soted_names=sorted(name_iterable,key=_sortkeyfunc)
        name_dict={}
        for key,group in itertools.groupby(sorted_names,_groupkeyfunc):
            name_dict[key]=tuple(group)
        return name_dict
    pieces_order={2:(-1,0),3:(-1,0,1)}
    def _sortkeyfunc(name):
        '''name是带有名和姓以及可选的中名或首字母的字符串
        这些部分之间用空格隔开;返回的字符串的顺序为姓-名-中名，
        以满足排序的需要"""
        name_parts=name.split()
        return ' '.join([name_parts[n] for n in pieces_order[len(name_parts)]])
    def _groupkeyfunc(name):
        '''返回键（即姓的首字母）被用于分组'''
        return name.split()[-1][0]

    #python2.3是实现
    def groupnames(name_iterable):
        name_dict={}
        for name in name_iterable:
            key=_groupkeyfunc(name)
            name_dict.setdefault(key,[]).append(name)
        for k,v in name_dict.iteritems():
            aux=[_sortkeyfunc(name),name for name in v]
            aux.sort()
            name_dict[k] = tuple([n for __,n in aux])
        return name_dict
