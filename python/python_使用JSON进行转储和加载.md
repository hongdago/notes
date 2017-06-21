使用JSON进行转储和加载
=========================
### python与jSON的类型映射
|Python类型|JSON|
|----------|:----|
|dict|object|
|list,tuple|array|
|str|string|
|int,float|number|
|True|true|
|False|false|
|None|null|
> 除了以上定义的几种类型，其他类型都不支持，并且必须使用扩展函数将其转换为以上的一种类型
* 示例

    ```
    import datetime
    from collections import defaultdict
    class Post:
        def __init__(self,date,title,rst_text,tags):
            self.date=date
            self.title=title
            self.rst_text=rst_text
            self.tags=tags
        def as_dict(self):
            return dict(
                date=str(self.date),
                title=self.title,
                underline="-"*len(self.title),
                rst_text=self.rst_text,
                tag_text=" ".join(self.tags)
            )
        """重构编码函数,添加特性"""
        @property
        def _json(self):
            return dict(
                __class__=self.__class__.__name__,
                __args__=[],
                __kw__=dict(
                    date=self.date,
                    title=self.title,
                    rst_text=self.rst_text,
                    tags=self.tags)
            )

    class Blog:
        def __init__(self,title,posts=None):
            self.title=title
            self.entries=posts if posts is not None else []
        def append(self,post):
            self.entries.append(post)
        def by_tag(self):
            tag_index=defaultdict(list)
            for post in self.entries:
                for tag in post.tags:
                    tag_index[tag].append(post.as_dict())
            return tag_index
        def as_dict(self):
            return dict(
                title=self.title,
                underline="="*len(self.title),
                entries=[p.as_dict() for p in self.entries]
            )

        """重构编码函数,添加特性"""
        @property
        def _json(self):
            return dict(
                __class__=self.__class__.__name__,
                __args__=[self.title,self.entries],
                __kw__={}
            )

    #自定义JSON编码函数
    def blog_encode(object):
        if isinstance(object,datetime.datetime):
            return dict(
                __class__="datetime.datetime",
                __args__=[],
                __kw__=dict(
                    year=object.year,
                    month=object.month,
                    day=object.day,
                    hour=object.hour,
                    minute=object.minute,
                    second=object.second)
                )
        elif isinstance(object,Post):
            return dict(
                __class__="Post",
                __args__=[],
                __kw__=dict(
                    date=object.date,
                    title=object.title,
                    rst_text=object.rst_text,
                    tags=object.tags)
                )
        elif isinstance(object,Blog):
            return dict(
                __class__="Blog",
                __args__=[object.title,
                    object.entries],
                __kw__={}
            )
        else:
            return json.JSONEncoder.default(object)

    #重构JSON编码
    def blog_encode2(object):
        if isinstance(object,datetime.datetime):
            return dict(
                __class__="datetime.datetime",
                __args__=[],
                __kw__=dict(
                    year=object.year,
                    month=object.month,
                    day=object.day,
                    hour=object.hour,
                    minute=object.minute,
                    second=object.second)
                )
        else:
            try:
                return object._json
            except AttributeError:
                return json.JSONEncoder.default(object)


    #自定义JSON解码
    def blog_decode(some_dict):
        if set(some_dict.keys()) == set(["__class__","__args__","__kw__"]):
            class_ = eval(some_dict['__class__'])
            return class_(*some_dict['__args__'],**some_dict['__kw__'])
        else:
            return some_dict

    if __name__=='__main__':
        travel=Blog("Travel")
        travel.append(Post(date=datetime.datetime(2017,6,20,16,35),
            title="Hard Aground",
            rst_text="""Some embarrassing revelation.
             Includeing ☻ and ⚓""",,
            tags=("#RedRanger","#Whitby42","#ICW")
        ))
        travel.append(Post(date=datetime.datetime(2017,6,22,13,35),
            title="Anchor Follies",
            rst_text="""Some witty epigram.
             Includeing < & > characters.""",
            tags=("#RedRanger","#Whitby42","#Mistakes")
        ))

        #使用Jinja2打印Blog对象
        from jinja2 import Template
        blog_template=Template("""
            {{title}}
            {{underline}}

            {% for e in entries %}
            {{e.title}}
            {{e.underline}}

            :date: {{e.date}}

            :tags: {{e.tag_text}}
            {% endfor %}
            Tag Index
            ===========
            {% for t in tags %}
            * {{t}}
              {% for post in tags[t] %}

              - '{{post.title}}'_
              {% endfor %}
            {% endfor %}
        """)
        print(blog_template.render(tags=travel.by_tag(),**travel.as_dict()))

        #使用json打印BLog对应的dict
        import json
        print(json.dumps(travel.as_dict(),indent=4))

        #使用自定义编码函数打印Blog对象
        text=json.dumps(travel,indent=4,default=blog_encode)
        print(text)

        #使用自定义解码函数恢复Blog对象
        blog=json.loads(text,object_hook=blog_decode)
        print(blog_template.render(tags=blog.by_tag(),**blog.as_dict()))

        #将JSON写入文件
        with open("temp.json","w",encoding="UTF-8") as target:
            json.dump(travel,target,separators=(',',':'),default=blog_encode2)
    
        #从文件中回复Blog对象
        with open("temp.json","r",encoding="UTF-8") as source:
            object=json.load(source,object_hook=blog_decode)
            print(object.as_dict())
    ```
