python_使用XML转储和加载.md
=============================
当考虑将一个python对象转储为XML文档是，以下有3种常用的方法可以用来创建文本:
* 在类的设计中包括XML输出方法。通过使用这种方法，我们类生成的字符串就直接进入XML文档中
* 使用xml.etree.ElementTree创建ElementTree节点并且返回这个结构。
* 使用一个外部的模板并将属性写入模板中

### 使用字符串模板转储对象
将python对象序列化为XML的一种方式是创建XML文本。这种方式有两个缺陷
* 忽略XML命名空间，还需稍微对文本进行改动来生成相应的标签
* 每个类还需要适当地将<、&、>和‘’字符相应的转化为XML中的&lt;、&gt;、&anmp;和&quot;。html模块的html.escape()函数可以完成这类转换
>示例
    
    import datetime
    class Post:
        def __init__(self,date,title,rst_text,tags):
            self.date=date
            self.title=title
            self.rst_text=rst_text
            self.tags=tags
        def xml(self):
            tags="".join("<tag>{0}</tag>".format(t) for t in self.tags)
            return """
                <entry>
                    <title>{0.title}</title>
                    <date>{0.date}</date>
                    <rst_text>{0.rst_text}</rst_text>
                    <tags>{1}</tags>
                </entry>
                """.format(self,tags)
    class Blog:
        def __init__(self,title,posts=None):
            self.title=title
            self.entries=posts if posts is not None else []
        def append(self,post):
            self.entries.append(post)
        def xml(self):
            children="\n".join(c.xml() for c in self.entries)
            return """
                <blog><title>{0.title}</title><entries>{1}</entries></blog>
                """.format(self,children)
    if __name__=='__main__':
        travel=Blog("Travel")
        travel.append(Post(date=datetime.datetime(2017,6,20,16,35),
            title="Hard Aground",
            rst_text="""Some embarrassing revelation.
             Includeing ☻ and ⚓""",
            tags=("#RedRanger","#Whitby42","#ICW")
        ))
        travel.append(Post(date=datetime.datetime(2017,6,22,13,35),
            title="Anchor Follies",
            rst_text="""Some witty epigram.
             Includeing < & > characters.""",
            tags=("#RedRanger","#Whitby42","#Mistakes")
        ))
        print(travel.xml())

### 使用xml.etree.ElementTree转储对象
>示例
    
    import datetime
    import xml.etree.ElementTree as XML
    class Post:
        def __init__(self,date,title,rst_text,tags):
            self.date=date
            self.title=title
            self.rst_text=rst_text
            self.tags=tags
        def xml(self):
            post=XML.Element("entry")
            title=XML.SubElement(post,"title")
            title.text=self.title
            date=XML.SubElement(post,"date")
            date.text=str(self.date)
            tags=XML.SubElement(post,"tags")
            for t in self.tags:
                tag=XML.SubElement(tags,"tag")
                tag.text=t
            text=XML.SubElement(post,"rst_text")
            text.text=self.rst_text
            post.tail="\n"
            return post
    class Blog:
        def __init__(self,title,posts=None):
            self.title=title
            self.entries=posts if posts is not None else []
        def append(self,post):
            self.entries.append(post)
        def xml(self):
            blog=XML.Element("blog")
            title=XML.SubElement(blog,"title")
            title.text=self.title
            title.tail="\n"
            entries=XML.SubElement(blog,"entries")
            entries.extend(c.xml() for c in self.entries)
            blog.tail="\n"
            return blog
    if __name__=='__main__':
        travel=Blog("Travel")
        travel.append(Post(date=datetime.datetime(2017,6,20,16,35),
            title="Hard Aground",
            rst_text="""Some embarrassing revelation.
             Includeing ☻ and ⚓""",
            tags=("#RedRanger","#Whitby42","#ICW")
        ))
        travel.append(Post(date=datetime.datetime(2017,6,22,13,35),
            title="Anchor Follies",
            rst_text="""Some witty epigram.
             Includeing < & > characters.""",
            tags=("#RedRanger","#Whitby42","#Mistakes")
        ))
        print(XML.tostring(travel.xml(),encoding="unicode))

### 加载XML文档
从一个XML文档中加载python对象分为两步，首先，我们需要对XML文本解析，用于创建文档对象，然后，需要对用于生成Python对象的文档对象进行检查。
> 示例

    import xml.etree.ElementTree as XML
    doc=XML.parse(io.StringIO(text.decode('utf-8')))
    xml_blog=doc.getroot()
    blog=Blog(xml_blog.findtext('title'))
    for xml_post in xml_blog.findall('entries/entry'):
        tags=[t.text for t in xml_post.findall('tags/tag')
        post=Post(date=datetime.datetime.strptime(xml_post.findtext('date'),'%Y-%m-%d %H:%M:%S'),
            title=xml_post.findtext('title'),
            tags=tags
            rst_text=xml_post.findtext('rst_text')
        )
        blog.append(post)
