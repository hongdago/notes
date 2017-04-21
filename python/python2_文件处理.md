文件处理(python2)
=====================
### 搜索和替换文件中的文本
    
    #!/usr/bin/python
    import os , sys
    nargs=len(sys.argv)
    if not 3 <= nargs <=5:
        print "Usage: %s search_text replace_text [infile [outfile]]" % \
            os.path.basename(sys.argv[0])
    else:
        stext=sys.argv[1]
        rtext=sys.argv[2]
        input_file=sys.stdin
        output_file=sys.stdout
        if nargs>3:
            input_file = open(sys.argv[3])
        if nargs>4:
            output_file= open(sys.argv[4])
        for  s in input_file:
            output_file.write(s.replace(stext,rtext))
        output.close()
        input.close()

### 从文件中读取指定的行
* 使用Python标准库linecache

    ```
    import linecache
    theline=linecache.getline(thefilepath,desired_line_number)
    ```
* 针对大文件的处理
    
    ```
    def getline(thefilepath,desired_line_number):
        if desired_line_number < 1 : return ''
        for current_line_number,line in enumerate(open(thefilepath,'rU'):
            if current_line_number == desired_line_number-1:return line
        return ''
    ```

### 计算文件的行数
* 对于不大的文件，最简单的方法
    
    ```
    count=len(open(thefilepath,'rU').readlines())
    ```
* 对于大文件
    
    ```
    count=-1
    for count,line in enumerate(open(thefilepath,'rU')):
        pass
    return count+1
    ```

### 处理文件中的每个词
    
    #定义方法，如何迭代所有的元素
    def word_of_file(thefilepath,line_to_words=str.split):
        the_file=open(thefilepath)
        for line in the_file:
            for word in line_to_words(line):
                yield word                     #定义生成器
        the_file.close()
    #处理每个元素
    for word in word_of_file(thefilepath):
        dosomethingwith(word)

    #对word_of_file的进一步封装
    import re
    def word_by_re(thefilepath,repattern=r"[\w'-]+"):
        wre=re.compile(repattern)
        def line_to_words(line):
            for mo in wre.finditer(line):
                return mo.group(0)
        return word_of_file(thefilepath,line_to_words)

### 更新随机存取文件

    import struct
    format_string = '8l' #假定一条记录是8个4字节整数
    thefile=open('somebindfile','r+b')
    record_size=struct.calcsize(format_string)
    record_number = 4    #假定需要修改的是第五条数据
    thefile.seek(record_size * recode_number)
    buffer=thefile.read(record_size)
    fields=list(struct.unpack(format_string,buffer))
    #进行计算，并修该相关字段
    domeSomething(fields)
    #然后
    buffer=struct.pack(format_string,*fields)
    thefile.seek(record_size.record_number)
    thefile.write(buffer)
    thefile.close()

### 从zip文件中读取数据
    
    import zipfile
    z=zipfile.ZipFile("zipfile.zip","r")  #r标志可以应付所有平台下的zip文件
    for filename in z.namelist():
        print "File:", finename
        bytes=z.read(filename)
        print 'has',len(bytes),'bytes'

    #创建一个zip文件，并从中导入一个模块，并删除
    import zipfile,tempfile,os,sys
    handle,filename=tempfile.mkstemp('.zip')
    os.close(handle)
    z=zipfile.ZipFile(filename,'w')
    z.writestr('hello.py','def f(): return "Hello world from" + __file__\n')
    z.close()
    sys.path.insert(0,filename)
    import hello
    print hello.f()
    os.unlink(filename)      #same as os.remove

### 处理字符串中的zip文件
* 程序接受到一个字符串，其内容是一个zip文件的内容，需要读取这个zip文件的信息

    ```
    import zipfile
    try:
        from cStringIO import StringIO
    except ImportError:
        from StringIO import StringIO
    class ZipString(zipfile.ZipFile):
        def __init__(self,datastring):
            zipfile.ZipFile(self,StringIO(dataString))
    ```

### 将文件树归档到一个压缩的tar文件
    
    import tarfile,os
    def make_tar(folder_to_backup,dest_folder,compression='bz2'):
        if compression:
            dest_ext = '.'+compression
        else:
            dest_ext= ''
        arcnmae=os.path.basename(folder_to_backup)
        dest_name='%s.tar%s' % (arcnmae,dest_ext)
        dest_path=os.path.join(dest_folder,dest_name)
        if compression:
            dest_cmp=":"+compression
        else:
            dest_cmp=''
        out=tarfile.TarFile.open(dest_path,'w'+dest_cmp)
        out.add(folder_to_backup,arcnmae)
        out.close()
        return dest_path

### 将二进制数据发送到windows的标准输出

    import sys
    if sys.platform == 'win32':
        import os,msvcrt
        msvrt.setmode(sys.stdout.fileno(),os.O_BINARY)


### 用类文件对象适配真实文件对象

    import tempfile,types
    CHUNK_SIZE=16*1024
    def adapt_file(fileObj):
        if isinstance(fileObj,file):return fileObj
        tmpFileobj=tempfile.TemporaryFile()
        while True:
            data=fileObj.read(CHUNK_SIZE)
            if not data: break
            tmpFileobj.write(data)
        fileObj.close()
        tmpFileobj.seek(0)
        return tmpFileobj

### 遍历目录树
    
    import os,fnmatch
    def all_files(root,patterns='*' ,single_level=False,yield_folders=False):
        #将模式从字符串中取出放入列表中
        patterns = patterns.split(";")
        for path,subdirs,files in os.walk(root):
            if yield_folders:
                files.extend(subdirs)
            files.sort()
            for name in files:
                for pattern in patterns:
                    if fnmatch.fnmatch(name,pattern):
                        yield os.path.join(path,name)
                        break
            if single_level:
                break

### 在目录树中改变文件扩展名

    import os
    def swapextensions(dir,before,after):
        if before[:1] != '.':
            before='.'+before
        thelen=-len(before)
        if after[:1] != '.':
            after='.'+after
        for path,subdir,files in os.walk(dir):
            for oldfile in files:
                if oldfile[thelen:] == before:
                    oldfile=os.path.join(path,oldfile)
                    newfile=oldfile[:thelen]+after
                    os.rename(oldfile,newfile)
    if __name__ =='__main__':
        import sys
        if len(sys.argv) != 4:
            print "Usage: swapext rootdir before after"
            sys.exit(1)
        swapextensions(sys.argv[1],sys.argv[2].sys.argv[3])

### 从指定的搜索路径寻找文件
    
    import os
    def search_file(filename,search_path,pathsep=os.pathsep):
        """给定一个搜索路径，根据请求的名字找到文件"""
        for path in search_path.split(os.pathsep):
            candidate=os.path.join(path,filename)
            if os.path.isfile(candidate):
                return os.path.abspath(candidate)
        return None
    if __name__=='__main__':
        search_path='/bin'+os.pathsep+"/usr/bin"
        find_file=search_path('ls',search_path)
        if find_file:
            print "File 'ls' found at %s  " % find_file
        else:
            print "File 'ls' not found"

### 根据指定的搜索路径和模式寻找文件

    import glob,os
    def all_files(pattern,search_path,pathsep=os.pathsep):
        """给定搜索路径，找出所有满足匹配条件的文件"""
        for path in search_path.split(pathsep):
            for match in glob.glob(os.path.join(path,pattern):
                yield match

### 在python的搜索路径中寻找文件

    import os,sys
    class Error(Exception):pass
    def _find(pathname,matchFunc=os.path.isfile):
        for dirname in sys.path:
            candidate=os.path.join(dirname,pathname)
            if matchFunc(candidate):
                return True
        raise Error("Can't find file %s " % pathname)
    def findFile(pathname):
        return _find(pathname)
    def findDir(dirname):
        return _find(dirname,matchFunc=os.path.isdir)

### 动态地改变Python搜索路径

    def addSysPath(new_path):
        """
        addSysPath(new_path):给Python的sys.path增加一个目录
        如果目录不存在或者已经在sys.path中，则不操作
        返回1表示成功，-1表示new_path不存在，0表示已经在sys.path中
        """
        import os,sys
        if not os.path.isdir(new_path):return -1
        new_path = os.path.abspath(new_path)
        is sys.platform == 'win32':
            new_path=new_path.lower()
        #检查当前所有路径:
        for x in sys.path:
            x=os.path.abspath(x)
            if sys.platform == 'win32':
                x=x.lower()
            if new_path in (x,x+os.sep):
                return 0
        sys.path.append(new_path)
        #如果想让new_path 在sys.path处于最前
        #sys.path.insert(0,new_path)
        return 1
    if __name__=='__main__':
        #测试，显示用法
        import sys
        print "Before":
        for x in sys.path: print x
        if sys.platform == 'win32':
            print addSysPath('c:\\Temp')
            print addSysPath('c:\\Temp')
        else:
            print addSysPath('/usr/lib/my_modules')
        print 'After':
        for x in sys.path: print x

### 计算目录间的相对路径

    import os,itertools
    def all_equal(elements):
        '''若所有元素都相等，则返回True,否则返回False'''
        first_element=elements[:1]
        for other_element in elements[1:]:
            if other_element != first_element:return False
        return True
    def common_prefiex(*sequences)
        '''返回所有序列开头部分共同元素的列表，紧接一个各序列的不同
        尾部列表'''
        #如果没有sequences,返回
        if not sequences: return [],[]
        #并行的循环序列
        comm=[]
        for elements in itertools.izip(*sequences):
            #若不是所有元素相等，跳出循环
            if not all_equal(elements):break
            comm.append(elements[0])
        #返回相同的头部和各自不同的尾部
        return comm,[sequence[len(comm):] for sequence in sequences)
    def relpath(p1,p2,sep=os.path.sep,pardir=os.path.pardir):
        '''返回p1对p2的相对路径
            特殊情况:空串,if p1 == p1
                     p2,如果p1 ,p2 完全没有相同元素'''
        comm,[u1,u2]=common_prefiex(p1.split(sep),p2.split(sep))
        if not comm:
            return p2
        return sep.join([pardir]*len(u1)+u2)
    def test(p1,p2,sep=os.path.sep):
        print "from",p1,"to",p2," --> ",relpath(p1,p2,sep)

### 从OpenOffice.org文档中提取文档
>OpenOffice.org 文档其实就是一个聚合了XML文件的zip文件

    import zipfile,re
    rx_stripxml=re.compile("<[^>]*?>",re.DOTALL|re.MULTILINE)
    def convert_OO(filename,want_text=True):
        """将一个OpenOffice.org文件转换成xml文本"""
        zf=zipfile.ZipFile(filename,"r")
        data=zf.read("context.xml")
        zf.close()
        if want_text:
            data=" ".join(rx_stripxml.sub(" ",data).split())
        return data

### 使用跨平台的文件锁

    import os
    if os.name=='nt':
        import win32con,win32file,pywintypes
        LOCK_HK=win32con.LOCKFILE_EXCLUSIVE_LOCK
        LOCK_SH=0
        LOCK_NB=win32con.LOCKFILE_FAIL_IMMEDIATELY
        __overlapped=pywintypes.OVERLAPPED()
        def lock(file,flags):
            hfile=win32file._get_osfhandle(file.fileno())
            win32file.LockFileEx(hfile,flages,0,0xffff0000,__overlapped)
        def unlock(file):
            hfile=win32file._get_osfhandle(file.fileno)
            win32file.UnlockFileEx(hfile,0,0xffff0000,__overlapped)
    elif os.name=='posix':
        from fcntl import LOCK_EX,LOCK_SH,LOCK_NB
        def lock(file,flag):
            fcntl.flock(file.fileno(),flags)
        def unlock(file):
            fcntl.flock(file.fileno(),fcntl.LOCK_UN)
    else:
        raise RuntimeError("PortaLocker only defined for nt and posix platforms")

### 带版本号的文件名

    def VersionFile(file_spec,vtype='copy'):
        import os,shutil
        if os.path.isfile(file_spec):
            #检查vtype参数
            if vtype not in ('copy','renmae'):
                raise ValueError,'Unknow vtype %r ' % (vtype)
            #确认根文件名，所以扩展名不会太长
            n,e=os.path.splitext(file_spec)
            #是不是一个以点为前导的三个数字
            if len(e) == 4 and e[1:].isdigit():
                num =1+int(e[1:])
                root =n
            else
                num=0
                root=file_spec
            #寻找下一个可用的文件版本
            for i in xrange(num,1000):
                new_file = "%s.%03d" % (root,i)
                if not os.path.exists(new_file):
                    if vtype=='copy':
                        shutil.copy(file_spec,new_file)
                    else:
                        os.rename(file_spec,new_file)
                    return True
            raise RuntimeError,"Can't %s%r，all names taken" % (vtype,file_spec)
        return False



