python_使用Shelve保存和获取对象.md
=====================================
>shelve模块没有直接支持原子性，它没有提供处理包括多个操作事务的方法。如果有包含多个操作的事务并且需要原子性，那么必须保证它们这些操作全部正常工作或者全部失败，这可能会用到更复杂的try:语句。当操作失败时，必须恢复为操作前的一个状态。

>shelve模块不保证所有类型的改变都可持久化，如果将一个可更改对象存在shelve上，然后在内存中改变对象的状态，持久化在shelve上的版本不会自动改变。如果希望改变已经存在shelve上的对象，必须显式地更新shevle。我们可以通过使用回写模式（writeback mode)让shelve对象追踪所有的变更，但是使用这个功能会影响性能。

### 创建shelve 
创建shelve用模块级别的函数shelve.open()来完成。shelve.open()函数需要两个参数：文件名和文件访问模式。通常用默认的'c'模式打开一个已经存在的shelve，或者当找不到制定的shevle时就创建一个新的。其它的模式用于一些特定的情况。
* 'r'是以只读的方式打开shelve
* 'w'必须指定一个已经存在的可读写的shelve,否则会抛异常
* 'n'创建一个新的空shelve,任何之前的版本都会覆盖

### 关闭shelve
关闭shelve是非常必要的，因为这样才能确保它被正确地写入磁盘中。shelve本身不是上下文管理器，但是可以用contextlib.closing()函数确保shevle被关闭。在一些情况下，可能也想显式地将shelve同步到磁盘，但是不关闭文件。shelve.sync()方法会在关闭之前保存改变。理想的生命周期代码会类似下面的代码：

    import shelve
    from contextlib import closing
    with closing(shelve.open('some_file')) as shelf:
        process(shelf)

### 为对象生成代理键
* 单一类的键: class:oid
* 当子类可以独立于父类存在时: Child.__class__:cid
* 当子类不能脱离父类存在时:Parent.__class__.pid:Child.__class__:cid

### 搜索、扫描和查询
* key.startswith("Parent:pid"):查询父对象和所有的子对象。
* key.startswith("Parent:pid:Child:):查找给定父对象的子对象
> 示例

    #删除父对象和它的所有子对象
    for obj in (shelf[k] for k in self.keys() if k.startswith(parent)):
        del obj

### 为shelve设计数据访问层

    import shelve
    """访问,为问题域中的所有对象提供统一的访问方法。"""
    class Access: 
        """第一部分，处理关闭和打开shelve文件"""
        def new(self,filename):
            self.database=shelve.open(filename,'n')
            self.max={'Post':0,'Blog':0}
            self.sync()
        def open(self,filename):
            self.database=shelve.open(filnename,'w')
            self.max=self.database['_DB:max']
        def close(self):
            if self.database:
                self.database['_DB:max']=self.max
                self.database.close()
            self.database=None
        def sync(self):
            self.database['_DB:max']=self.max
            self.database.sync()
        def quit(self):
            self.database.close()

        """第二部分，添加Blog和Post对象"""
        def add_blog(self,blog):
            self.max['Blog'] +=1
            key="Blog:{id}".format(id=self.max["Blog"])
            blog._id=key
            self.database[blog._id]=blog
            return blog
        def get_blog(self,id):
            return self.database[id]
        def add_post(self,blog,post):
            self.max['Post'] +=1
            try:
                key="{blog}:Post:{id}".format(blog=blog._id,id=self.max['Post'])
            except AttributeError:
                raise OPerationError("Blog not add")
            post._id=key
            post._blog=blog
            self.database[post._id]=post
            return post
        def get_post(self,id):
            return self.database[id]
        def replace_post(self,post):
            self.database[post._id]=post
            return post
        def delete_post(self,post):
            del self.database[post._id]

        """第三部分，对Blog和Post进行查询"""
        def __iter__(self):
            for k in self.database:
                if k[0] == '_':continue
                yield self.database[k]
        def blog_iter(self):
            for k in self.database:
                if not k.startswith('Blog'):continue
                if ":Post:" in k:continue # skip children Post
                yield self.database[k]
        def post_iter(self,blog):
            key="{blog}:Post".format(blog=blog._id)
            for k in self.database:
                if not k.startswith(key):continue
                yield self.database[k]
        def title_iter(self,blog,title):
            return (p for p in self.post_iter(blog) if p.title == title)

### 用索引提高性能

    import shelve
    class Access2(Access):
        def add_blog(self,blog):
            self.max['Blog'] +=1
            key="Blog:{id}".format(self.max['Blog'])
            blog._id=key
            blog._post_list=[]
            self.database[blog._id]==blog
            return blog

        def add_post(self,blog,post):
            self.max['Post'] +=1
            try:
                key="{blog}:Post:{id}".format(blog=blog._id,id=self.max['Post'])
            except AttributeError:
                raise OPerationError("Blog not add")
            post._id=key
            post._blog=blog
            self.database[post._id]=post
            blog._post_list.append(post._id)
            self.database[blog._id]=blog
            return post

        def delete_post(self,post):
            blog_id=post._blog._id
            del self.database[post._id]
            blog=self.database[blog_id]
            blog._post_list.remove(post._id)
            self.database[blog._id]=blog

        def __iter__(self):
            for k in self.database:
                if k[0] == "_" :continue
                yield self.database[k]
        def blog_iter(self);
            for k in self.database:
                if not k.startswith("Blog:"):continue
                if ":Post:" in k:continue # skip children Post
                yeild self.database[k]
        def post_iter(self,blog):
            for k in blog._post_list:
                yeild self.database[k]
        def title_iter(self,blog,title):
            return (p for p in self.post_iter(blog) if p.title==title)

### 创建顶层索引
    
    import shelve
    class Access3(Access2):
        def new(self, *args,**kw):
            super().new(*args,**kw)
            self.database['_DB:Blog']=[]
        
        def add_blog(self,blog):
            self.max["Blog"] +=1
            key="Blog:{id}".format(id=self.max["Blog"])
            blog._id=key
            blog._post_list=[]
            self.database[blog._id]=blog
            self.database['_DB:Blog].append(blog._id)
            return blog

        def blog_iter(self):
            return (self.database[k] for k in self.database['_DB:Blog'])

### 有关更多的索引维护工作

    class Access4(Access2):
        def new(self,*args,**kw):
            super().new(*args,**kw)
            self.database['_DB:Blog']=[]
            self.database['_DB:Blog_Title']=defaultdict(list)

        def add_blog(self,blog):
            self.max['Blog']+=1
            key="Blog:{id}".format(id=self.max['Blog'])
            blog._id=key
            blog._post_list=[]
            self.database[blog._id]=blog
            self.database['_DB:Blog'].append(blog._id)
            blog_title=self.database['_DB:Blog_Title']
            blog_title[blog.title].append(blog._id)
            self.database['_DB:Blog_Title']=blog_title
            return blog

        def update_blog(self,blog):
            """replace this blog,update index."""
            self.database[blog._id]=blog
            blog_title=self.database["_DB:Blog_Title"]
            #remove key fro index in olg spot.
            empties=[]
            for k in blog_title:
                if blog._id in blog_title[k]:
                    blog_title[k].remobe(blog._id)
                    if len(blog_title[k])==0 empties.append(k)
            #clean up zero-length lists form defaultdict
            for k in empties:
                del blog_title[k]
            #put key into index in new spot:
            blog_title[blog.title].append(blog._id)
            self.database['_DB:Blog_Title']=blog_title

        def blog_iter(self):
            return (self.database[k] for k in self.database['_DB:Blog'])

        def title_iter(self,title):
            blog_title=self.database['_DB:Blog_Title']
            return (self.database[k] for k in blog_title[title])

