python_用SQLite保存和获取对象.md
==================================
在使用SQL数据库时，在设计时需要做一些抉择。其中最重要的决定也许是如何最大化地解决阻抗不匹配问题，也就是如何解决从旧的SQL数据映射为python模型，有3中常见的策略。
* 完全不考虑Python的映射：这意味着我们不对数据库进行复杂的Python对象的查询，工作范围在一个完全独立的SQL框架中进行，这个框架包括原子数据元素和函数处理。使用这种方式就不必非常执着于为数据库对象持久化使用面向对象编程进行设计。我们只能使用4种基本的SQLite类型:NULL、INTEGER、REAL和TEXT,还有python中的datetime.date和datetime.datetime
* 手动映射：在类与SQL逻辑模型的表、列、行和键之间添加一个数据访问层
* ORM层:下载安装一个ORM层，用于完成在类和SQL逻辑模型的映射

### SQLite与Python的数据类型映射
|SQLite|python|
|------|:-----|
|NULL|None|
|INTEGER|int|
|REAL|float|
|TEXT|str|
|BLOB|bytes|

### SQLite创建库和表结构,DML,DCL
>示例
    
    import sqlite3
    SQL_DDL="""
        CREATE TABLE BLOG(
            ID INTEGER PRIMARY KEY AUTOINCREMENT,
            TITLE TEXT);
        CREATE TABLE POST(
            ID INTEGER PRIMARY KEY AUTOINCREMENT,
            DATE TIMESTAMP,
            TITLE TEXT,
            RST_TEXT TEXT,
            BLOG_ID INTEGER REFERENCES BLOG(ID));
        CREATE TABLE TAG(
            ID INTEGER PRIMARY KEY AUTOINCREMENT,
            PHRASE TEXT UNIQUE ON CONFLICT FAIL);
        CREATE TABLE ACCOC_POST_TAG(
            POST_ID INTEGER REFERENCES POST(ID),
            TAG_ID INTEGER REFERENCES TAG(ID));
        """
    #创建数据库
    database=sqlite3.connect('p2_c11_blog.db')
    database.executescript(SQL_DDL)  #执行建表DDL语句

    #进行插入(INSERT)操作
    init_blog="""
        INSERT INTO BLOG(TITLE) VALUES(?)
    """
    database.execute(init_blog,("Travel Blog",))

    #进行更新(UPDATE)操作
    update_blog="""
        UPDATE BLOG SET TITLE=:new_title WHERE TITLE=:old_title
    """
    database.execute("BEGIN")
    database.execute(update_blog,dict(new_title="2013-14 Travel"
        ,old_title="Travel Blog"))
    database.commit()

    #进行delete操作
    delete_post_tag_by_blog_title="""
        DELETE FROM ASSOC_POST_TAG
        WHERE POST_ID IN (
            SELECT DISINCT POST_ID
            FROM BLOG JOIN POST ON BLOG.ID = POST.BLOG_ID
            WHERE BLOG.TITLE=:old_title)
    """
    delete_post_by_blog_title="""
        DELETE FROM POST WHERE BLOG_ID IN (
            SELECT ID FROM BLOG WHERE TITLE=:old_title)
    """
    delete_blog_by_title="""
        DELETE FROM BLOG WHERE TITLE=:old_title
    """
    try:
        with database: #事务处理
            title=dict(old_title="2013-2014 Travel")
            database.execute(delete_post_tag_by_blog_title,title)
            database.execute(delete_post_by_blog_title,title)
            database.execute(delete_blog_by_title,title)
        print("Delete finished normally."
    except Exception as e:
        print("Rolled Back due to {0}".fomat(e))

    #执行查询(select)操作
    query_blog_by_title="""
        SELECT * FROM BLOG WHERE TITLE=?
    """
    for blog in database.execute(query_blog_by_title,("2013-2014 Travel",)):
        print(blog[0],blog[1])

    """为了完全遵循Python中DB-API的标准，可以分解为如下几步:
        cursor=database.cursor()
        cursor.execute(query_blog_by_title,("2013-2014 Travel",))
        for blog in cursor.fetchall():
            print(blog[0],blog[1])

### SQLite事务处理
SQLite的隔离级别：
* isolation_level=None:默认的设置，也被称为自动提交(autocommit)模式
* isolation_level='DEFERRED':在这种模式下，事务中的锁添加的越晚越好。例如BEGIN语句，并没有立即获得任何锁。对于其他的读操作可以获得共享锁，写操作将获得保留的锁。然而这样可以最大化并发，但是在多个进程中也会产生死锁
* isolation_level='EXCLUSIVE':在这种模式下，事务的BEGIN语句会获得一个锁，阻止其他操作的访问。
> 事务执行的示例

    """
    当建立数据库链接时我们将隔离级别设置为DEFERRED,这意味着我们需要显式
    地开始和结束没一个事务。
    """
    database=sqlite3.connect('database.db',isolation_level='DEFERRED')
    try:
        database.execute('BEGIN')
        database.execute("some statement")
        database.execute("another statement")
        database.commit()
    except Exception as e:
        database.rollback()
        raise e

    """
    同上面类似的，我们可以将sqlite3.Connection对象作为一个上下文来
    简化
    """
    database=sqlite3.connect('database.db',isolation_level='DEFERRED')
    with database:
        database.execute("some statement")
        database.execute("another statement")

### 从Python对象到SQLite 数据的映射
转换和适配
> 示例

    import decimal

    def adapt_currency(value):
        return str(value)
    """
    完成将decimal.Decimal 对象适配为数据库适当的形式
    """
    sqlite3.register_adapter(decimal.Decimal,adapt_currency) #

    def convert_currency(bytes):
        return decimal.Decimal(bytes.decode())
    """
    用于从SQLite字节对象转换为Python中的decimal.Decimal对象
    """
    sqlites.register_converter("DECIMAL",convert_currency)`

    """
    一旦定义了适配器和转换器，就能将DECIMAL看作一种被完全支持
    的列类型。除此之外，在建立数据库链接时通过设置detect_types=sqlite3.PARSE_DECLTYPES来通知SQLite
    """
    decimal_DDL="""
            CREATE TABLE BUDGET（
                year INTEGER,
                month INTEGER,
                category TEXT,
                amount DECIMAL
            ）
    """
    database=sqlite3.connect("database.db",detect_type=sqlite3.PARSE_DECLTYPES)
    database.exceute(decimal_ddl)

    insert_budget="""
        INSERT INTO BUDGET(year,month,category,amount) values(:year,
        :month,:category,:amount)
    """
    database.execute(insert_budget,dict(year=2013,month=1,category="fuel",
        amount=decimal.Deciaml('256.78')))

### 手动完成python对象到数据库中行的映射
> 示例
    
    import datetime
    import sqlite3
    from collections import defaultdict

    class TooManyValues(Exception):
        pass

    class Blog:
        def __init__(self,**kw):
            self.id=kw.pop('id',None)
            self.title=kw.pop('title',None)
            if kw:
                raise TooManyValues(kw)
            self.entries=[]
        def append(self,post):
            self.entries.append(post)
        def by_tag(self):
            tag_index=defaultdict(list)
            for post in self.entries:
                for tag in post.tags:
                    tag_index[tag].append(post)
            return tag_index
        def as_dict(self):
            return dict(title=self.title,
                underline="="*len(self.title),
                entries=[p.as_dict for p in self.entries]
            }

        @property
        def entries(self):
            return self._access.post_iter(self)

    class Post:
        def __init__(self,**kw):
            self.id=kw.pop('id',None)
            self.date=kw.pop('date',None)
            self.title=kw.pop('title',None)
            self.rst_text=kw.pop('rst_title',None)
            self.tags=list()
            if kw:
                raise TooManyValues(kw)
        def append(self,tag):
            self.tags.append(tag)
        def as_dict(self0：
            return dict(date=str(self.date),
                title=self.title,
                underline="-"*len(self.title),
                rst_text=self.rst_text,
                tag_text=" ".join(self.tags)
            )

    """为SQLite设计一个访问层"""
    class Access:
        get_last_id="SELECT last_insert_rowid()"
        def open(self,filename):
            self.database=sqlite3.connect(filename)
            self.database.row_factory=sqlite3.Row
        def get_blog(self,id):
            query_blog="""
                SELECT * from BLOG WHERE ID= ？
            """
            row=self.database.execute(query_blog,(id,)).fetchone()
            blog=Blog(id=row['ID'],title=row['TITLE'])
            blog._access=self  #实现容器的关系
            return blog
        def add_blog(self,title):
            insert_blog="""
                INSERT INTO BLOG (TITLE) value (:title)
            """
            self.database.execute(insert_blog,dict(title=title))
            row=self.database.execute(get_last_id).fetchone()
            blog=Blog(id=row[0],title=tile)
            blog._accsss=self
            return blog

        def get_post(self,id):
            query_post="""
                SELECT * FROM POST WHERE ID = ?
            """
            row=self.database.execute(query_post,(id,)).fetchone()
            post=Post(id=row['ID'],title=row['TITLE'],date=row['DATE']
                ,rst_text=row['RST_TEXT']
            query_tags="""
                SELECT TAG.* 
                FROM TAG JOIN ASSOC_POST_TAG ON TAG_ID=ASSOC_POST_TAG.TAG_ID
                WHERE ASSOC_POST_TAG.POST_ID=?
            """
            results=self.database.execute(query_tags,(id,))
            for id,tag in results:
                post.append(tag)
            post._access=self
            return post

        def add_post(self,blog,post):
            insert_post="""
                INSERT INTO POST(TITLE,DATE,RST_TEXT,BLOG_ID)
                    VALUES(:title,:date,:rst_text,:blog_id)
            """
            query_tag="""
                SELECT * FROM TAG WHERE PHRASE=?
            """
            insert_tag="""
                INSERT INTO TAG(PHRASE) VALUES (?)
            """
            insert_association="""
                INSERT INTO ASSOC_POST_TAG(POST_ID,TAG_ID) 
                    VALUES (:post_id,:tag_id)
            """
            with self.database:
                self.database.execute(insert_post,
                    dict(title=post.title,date=post.date,
                        rst_text=post.rst_text,blog_id=blog.id
                    )
                row = self.database.execute(get_last_id).fetchone()
                post.id=row[0]
                for tag in post.tags:
                    tag_row = self.database.execute(query_tag,(tag,)).fetchone()
                    if tag_row is not None:
                        tag_id=tag_row['ID']
                    else:
                        self.database.execute(insert_tag,(tag,))
                        row=self.database.execute(get_last_id).fetchone()
                        tag_id=row[0]
                    self.database.execute(insert_association,
                        dict(tag_id=tag_id,post_id=post.id)
            post._access=self
            return post

            def blog_iter(self):
                query="""
                    SELECT * FROM BLOG
                """
                results=self.database.exceute(query)
                for row in results:
                    blog=Blog(id=row['ID'],title=row['TITLE'])
                    yield blog
            def post_iter(self,blog):
                query="""
                    SELECT * from POST WHERE BLOG_ID=?
                """
                results = self.database.execute(query,(blog.id,))
                for row in results:
                    yield self.get_post(row['ID'])
