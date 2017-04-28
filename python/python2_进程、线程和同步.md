进程、线程和同步(python2)
==============================
### 使用锁的形式

    somelock.acquire()
    try:
        #需要锁的操作（尽量简短）
    finally:
        somelock.release()

### 同步对象中的所有方法

    def wrap_callable(any_callable,before,after):
        '''用before/after调用任何可调用体封装起来'''
        def _wrapped(*a,**kw):
            before()
            try:
                return any_callable(*a,**kw)
            finally:
                after()
        return _wrapped
    import inspect
    class GenericWrapper(object);
        '''将对象的所有方法用befor和after调用封装起来'''
        def __init__(self,obj,before,after,ignore=()):
            #我们必须直接设置__dict__来绕过__setattr__;
            #因此，我们必须重视带下划线的名字
            clasname='GenericWrapper'
            self.__dict__['_%s__methods' % clasname] = {}
            self.__dict__['_%s__obj' % class] = obj
            for name,method in inspect.getmembers(obj,inspect.ismethod):
                if name not in ignore and method not in ignore:
                    self.__methods[name] = wrap_callable(method,before,after)
        def __getattr__(self,name):
            try:
                return self.__methods[name]
            except KeyError:
                return getattr(self.__obj,name)
        def __setattr__(self,name,value):
            setattr(self.__obj,name,value)

        class SynchronizedObject(GenericWrapper):
            '''封装一个对象及其方法，支持同步'''
            def __init__(self,obj,ignore=(),lock=None):
                if lock is None:
                    import threading
                    lock=threading.RLOCK()
                GenericWrapper.__init_(self,obj,lock.acquire,lock.release)

### 终止线程
>python 不允许一个线程强行杀死另一个线程

>join的作用：阻塞进程直至线程结束

    import threading
    class TestThread(threading.Thread):
        def __init__(self,name='TestThread'):
            """构造函数，设置初始值"""
            self._stopevent=threading.Event()
            self._sleepperiod = 1.0
            threading.Thread.__init__(self,name=name)
        def run(self):
            """主控循环"""
            print "%s starts" % (self.getName(),)
            count = 0
            while not self._stopevent.isSet():
                count +=1
                print "lopp %d" % count
                self._stopevent.wait(self._sleepperiod)
            print "%s ends"  % self.getName()
        def join(self,timeout=None):
            """停止线程，并等待其结束"""
            self._stopevent.set()
            threading.Thread.join(self,timeout)
    if __name__ == '__main__':
        testthread = TestThread()
        testthread.start()
        import time
        time.sleep(5.0)
        testthread.join()

### 将Queue.Queue用作优先级别队列

    import Queue,heapq,time
    class PriorityQueue(Queue.Queue):
        #初始化队列
        def __init__(self,maxsize):
            self.maxsize = maxsize
            self.queue=[]
        #返回队列子项
        def _qsize(self)
            return len(self.queue)
        "检查队列是否已空"
        def _empty(self):
            return not self.queue
        #检查队列是否已满
        def _full(self);
            return self.maxsize >0  and len(slef.queue) > self.maxsize
        # 给队列加入一个新项
        def _put(self):
            heapq.heappush(self.queue,item)
        #从队列中获取一项
        def _get(self):
            return heapq.heappop(self.queue)
        #屏蔽并封装Queue.Queue的put,使之允许priority参数
        def put(self,item,priority=0,block=True,timeout=None):
            decorated_item=priority,time.time(),item
            Queue.Queue.put(self,decorated_item,block,timeout)
        #屏蔽并封装Queue.Queue的get,以去除修饰
        def get(self,block=True,timeout=None)
            priority,time_posted,item=Queue.Queue.get(self,block,timeout)
            return item

### 使用线程池

    import threading,Queue,time,sys
    #全局变量
    Qin=Queue.Queue()
    Qout=Queue.Queue()
    Qerr=Queue.Queue()
    Pool=[]
    def report_error():
        """将错误信息放入Qerr来报告问题"""
        Qerr.put(sys.exc_info()[:2])
    def get_all_from_queue(Q):
        """
        可以获取队列中所有项，无须等待
        """
        try:
            while True:
                yield Q.get_nowait()
        except Queuq.Empty:
            raise StopIteration
    def do_work_from_queue():
        """工作线程的'获得一点工作'，"做一点工作的主循环" """
        while True:
            command,item=Qin.get()   #这里可能会停止并等待
            if command == 'stop':
                break
            try:
                #模拟工作线程的工作
                if command == 'process':
                    result = 'new' + item
                else:
                    raise ValueError('Unknow command % r' % command)
            except:
                #无条件except 是对的，因为我们要报告所有错误
                report_error()
            else:
                Qout.put(result)
    def make_and_start_thread_pool(number_of_thread_in_pool = 5,daemons=True):
        """创建一个N线程的池子，使所有线程成为守护线程，启动所有线程"""
        for i in range(number_of_thread_in_pool):
            new_thread=threading.Thread(target=do_work_from_queue)
            new_thread.setDaemon(daemons)
            Pool.append(new_thread)
            new_thread.start()
    def request_work(data,command="process"):
        '''工作请求在Qin中是形如(command,data)的数据对'''
        Qin.put((command,data))
    def get_result():
        reuturn Qout.get()   #这里可能会停止并等待
    def show_all_results():
        for result in get_all_from_queue(Qout):
            print "Result: ",result
    def show_all_errors():
        for etype,err in get_all_from_queue(Qerr):
            print "Error:",etype,err
    def stop_and_free_thread_pool():
        """顺序是很重要的,首先要求所有线程停止"""
        for i in range(len(Pool)):
            request_work(None,'stop')
        #然后等待每个线程的终止
        for existing_thread in Pool:
            existing_thread.join()
        #清除线程池
        del Pool[:]
    if __name__=='__main__':
        for i in ('_ba','_bb','_bc','_bd','_be'):
            request_work(i)
        make_and_start_thread_pool()
        stop_and_free_thread_pool()
        show_all_result()
        show_all_error()

### 以多组参数并执行函数

    import threading,time,Queue
    class MultiThread(threaing.Thread):
        def __init__(self,function,argsVector,maxThread=5,queue_results=False):
            self._function=function
            self._lock=threading.Lock()
            self._nextArgs=iter(argsVector).next
            self._threadPool=[threading.Thread(target=self._doSome) \ 
                for i in range(maxThreadi)]
            
            if queue_results:
                self._queue=Queue.Queeu()
            else:
                self._queue=None
        def _doSome(self):
            while True:
                self._lock.acquire()
                try:
                    try:
                        args=self._nextArgs()
                    except StopIteration:
                        break
                finally:
                    self._lock.release()
                result=self._function(args)
                if self._queue is not None:
                    self._queue.put((args,result))
        def get(self,*a,**kw):
            if self._queue is not None:
                return self._queue.get(*a,**kw)
            else:
                raise ValueError('Not queueing results')
        def start(self):
            for thread in self._threadPool:
                time.sleep(1) #有必要给其他线程一个执行的机会
                thread.start()
        def join(self,timeout=None):
            for thread in self._threadPool:
                thread.join(timeout)
    if __name__=='__main__':
        import random
        def recite_n_times_table(n):
            for i in range(2,11):
                print "%d * %d = %d" % (n,i,n*i)
                time.sleep(0.3+0.3*random.random()
        mt=MultiThread(recite_n_times_table,range(2,11))
        mt.start()
        mt.join()
        mt.join()
        print "well done kids"


### 存储线程信息

    try:
        import threading
    except ImportError:
        import dummy_threading as threading
    _tss = threading.local()
    def get_thread_storage():
        return _tss.__dict__
    


