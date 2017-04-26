调试和测试(python2)
======================
### 在linux上测试内存使用

    import os
    '''基于/proc伪文件编写测试文件'''
    _proc_statue='/proc/%d/status' % os.getpid()
    _scale={'kB':1024.0,'mB':1024.0*1024.0,'KB':1024.0,'MB':1024.0*1024.0}

    def _VmB(VmKey):
        '''给定VmKey字符串，返回字节数'''
        #获得伪文件/proc/<pid>/status
        try:
            t=open(_proc_statue)
            v=t.read()
            t.close()
        except IOError:
            return 0.0  #no-linux
        #获得VmKey行，如VmRSS: 999 kB\n
        i=v.index(VmKey)
        v=v[i:].split(None,3) #依照空白符切割
        if len(v) <3:
            return 0.0 #无效格式
        #将Vm装换成字节
        return float(v[1]) * _scale[v[2]]
    def memory(since=0.0):
        """返回虚拟内存使用的字节数"""
        return _VmB('VmSize:') -since
    def resident(since=0.0):
        """返回常驻内存使用的字节数"""
        return _VmB('VmRSS:')-since
    def stacksize(since=0.0):
        """返回栈使用的字节数"""
        return _VmB('VmStk:')-since

### 调用垃圾回收进程

    import gc
    def dump_garbage():
        """展示垃圾都是什么dongxi"""
        #强制收集
        print "\nGARBAGE:"
        gc.collect()
        print "\nGARBAGE OBJECTS:"
        for x in gc.garbage:
            s=str(x)
            if len(s) > 80: s=s[:77]+"..."
            print type(x),"\n ",s
    if __name__=='__main__':
        gc.enable()
        gc.set_debug(gc.DEBUG_LEAK)
        #模拟一个泄漏(一个引用自身的列表)
        l=[]
        l.append(l)
        del l
        dump_garbage()

### 捕获和记录异常

    import cStringIO,traceback
    def process_all_files(all_filenames,fatal_exception=(KeyboardInterrupt,MemoryError):
        bad_filenames={}
        for one_filename in all_filenames:
            try:
                process_one_file(one_filename)
            except fatal_exception:
                raise
            except Exception:
                f=cStringIO.StringIO()
                traceback.print_exc(file=f)
                bad_filenames[one_filename]=f.getvalue()
        return bad_filenames

### 在调试模块中跟踪表达式和注释

    import sys,traceback
    traceOutput = sys.stdout
    watchOutput = sys.stdout
    rawOutput   = sys.stdout
    watch_format=('File "%(fileName)s",line %(lineNumber)d, in %(methodName)s\n %(varName)s <%(varType)s> = %(value)s\n\n'
    def watch(variableName):
        if __debug__:
            stack=traceback.extract_stack()[-2:][0]
            actualCall=stack[3]
            if actualCall is None:
                actualCall = "watch([unknow])"
            left = actualCall.find('(')
            right= actualCall.find(')')
            paramDict = dict(varName=actualCall[left+1:right].strip(),
                            varTyep=str(type(variableName))[7:-2],
                            value=repr(variableName),
                            methodName=stack[2],
                            lineNumber=stack[1],
                            fileName=stack[0])
            watchOutput.write(watch_format % paramDict)
    trace_format=('File "%(fileName)s",line %(lineNumber)d,in %(methodName)s \n %(text)s\n\n')
    def trace(text):
        if __debug__:
            stack=traceback.extract_stack()[-2:][0]
            paramDict=dict(text=tet,
                        methodName=stack[2],
                        lineNumber=stack[1],
                        fileName=stack[0])
            traceOutput.write(trace_format % paramDict)
    def raw(text0;
        if __debug__:
            rawOutput.write(text)

### 从traceback中获取更多的信息

    import sys,traceback
    def print_exc_plus():
        """打印通常的回溯信息，且附有每帧中局部变量的列表"""
        tb=sys.exc_info()[2]
        while tb.tb_next:
            tb=tb.tb_next
        stack = []
        f=tb.tb_frame
        while f:
            stack.append(f)
            f=f.f_back
        stack.reverse()
        traceback.print_exc()
        print "Locals by frame,innermost last"
        for frame in stack:
            print 
            print "Frame %s in %s at line %s " % (frame.f_code.co_name,
                                                frame.f_code.co_filename,
                                                frame.f_lineno)
        for key,value in frame.f_locals.items():
            print "\t%20s = " % key,
            try:
                print value
            except:
                print "<ERROR WHILE PRINTING VALUE>"

### 使用doctest和unittest

    def add(a,b):
        """将任意两个对象相加并返回和
        >>> add(1,2)
        3
        >>> add([1],[2])
        [1,2]
        >>> add([1],2)
        Traceback (most recent call last):
        TypeError: can only concatenate list (not "int") to list
        """
        return a+b
    if __name__ =='__main__':
        import doctest
        doctest.testmod()

### 在单元测试中检查区间
    
    import unittest
    class IntervalTestCase(unittest.TestCase):
        def failUnlessInside(self,first,second,error,msg=None):
            """如果first不在区间内，失败;
            区间由 second +- error 构成"""
            if not(second-error) < first < (second+error):
                raise self.failureException(msg or '%r != %r (+-%r)' % (first,second,error)
        def failIfInside(self,first,second,error,msg=None):
            """如果first在区间中，失败
            区间由 second +- error 构成"""
            if (second - error) < first <(second +error):
                raise self.failureException(msg or '%r == %r (+-%r)' % (first,second,error)
        assertInside = failUnlessInside
        assertNotInside =failIfInside
    if __name__=='__main__':
        class IntegerArithenticTestCase(IntervalTestCase):
            def testAdd(self):
                self.assertInside((1+2),3.3,0.5)
                self.assertInside(0+1 ,1.1,0.01)
            def testMultipy(self):
                self.assertNotInside((0*10),.1,.05)
                self.assertNotInside((5*8),40.1,.2)
        unittest.main()
            


