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
* 针对打文件的处理
    
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




