XML处理(python2)
=========================
### 检查XML的格式完好性

    from xml.sax.handler import ContentHandler
    from xml.sax import make_parser
    from glob import glob
    import sys
    def parsefile(filename):
        parser=make_parser()
        parser.setContentHandler(ContentHandler())
        parser.parse(filename)
    for arg in sys.argv[1:]:
        for filename in glob(arg):
            try:
                parsefile(filename)
                print "%s is well-formed" % filename
            except Exception,e:
                print "%s is NOT well-formed" % filename

### 计算文档中标签的个数

    from xml.sax.handler import ContentHandler
    import xml.sax
    class CountHandler(ContentHandler):
        def __init__(self):
            self.tags={}
        def startElement(self,name,attr):
            self.tags[name] =1 + self.tags.get(name,0)
    parser=xml.sax.make_parser()
    handler=CountHandler()
    parser.setContentHandler(handler)
    parser.parse("test.xml")
    tags=handler.tags.keys()
    tags.sort()
    for tag in tags:
        print tag,handler.tags[tag]


### 获得XML文档中的文本
    
    from xml.sax.handler import ContentHandler
    import xml.sax
    import sys
    class TextHandler(ContentHandler):
        def characters(self,ch):
            sys.stdout.write(ch.encode("Lantin-1")
    parser=xml.sax.make_parser()
    handler=TextHandler()
    parser.setContentHandler(handler)
    parser.parse("test.xml")

### 自动探测XML的编码

    import codecs,encodings
    """调用者会给这个类一个字符缓冲区，并要求我们转换缓冲
    或自动探测缓冲所用的编码"""
    #None表示一个可能的变化的字节（XML中的##）
    autodetect_dict={ #bytepattern      :name
                    (0x00,0x00,0xFF,0xFF) : ("ucs4_be"),
                    (0xFF,0xFF,0x00,0x00) : ("ucs4_le"),
                    (0xFF,0xFF,None,None) : ("utf16_be"),
                    (0xFF,0xFE,None,None) : ("utf16_le"),
                    (0x00,0x3C,0x00,0x3F) : ("utf_16_be"),
                    (0x3C,0x00,0x3F,0xFF) : ("utf_16_le"),
                    (0x3C,0x3F,0x78,0x6D) : ("utf_8"),
                    (0x4C,0x6F,0xA7,0x94) : ("EBCDIC")
                    }
    def autoDetectXMLEncoding(buffer):
        """buffer --> encoding_name
            缓冲字符串应该是至少4字节。
            如果不能探测出编码则返回None
            注意，encoding_name 可能不是一个安装的编码器
        """
        #更有效的实现不是一次性对整个缓冲解码
        #而是一次解码一个字符并寻找需要的字符，比较繁琐
        encoding = "utf_8"  #根据XML规范，这是默认的编码
                            #这段代码在默认基础上进行完善
                            #一旦失败，它就退回到上次设定
                            #的编码
        bytes=byte1,byte2,byte3,byte4=map(ord,buffer[0:4])
        enc_info = autodetect_dict.get(bytes,None)
        if not enc_info: # 再次尝试自动探测，移除可能的变化字节
            bytes=byte1,byte2,None,None
            enc_info=autodetect_dict.get(bytes)
        if enc_info:
            encoding=enc_info
        #尝试找到一个使用XML声明的更精确的编码
        secret_decoder_ring = codecs.lookup(encoding)[1]
        decoded,length = secret_decoder_ring(buffer)
        first_line = decoded.split("\n",1)[0]
        if first_line and first_line.startswith(u"<?xml"):
            encoding_pos = first_line.find(u"encoding")
            if encoding_pos !=-1:
                #寻找双银行
                quote_pos = first_line.find(u'"',encoding_pos)
                if quote_pos == -1 :#寻找单引号
                    quote_pos = first_line.find("'",encoding_pos)
                if quote_pos>-1:
                    quote_char = first_line[quote_pos]
                    rest = first_line[quote_pos+1:]
                    encodeing=rest[:rest.find(quote_char)]
        return encoding


### 将一个XML文档转化成Python对象数

    from xml.parsers import expat
    class Element(object):
        """一个解析出来的元素"""
        def __init__(self,name,attributes):
            #记录标签名及属性字典
            self.name = name
            self.attributes = attributes
            #将元素的cdata和子元素清空
            self.cdata=''
            self.children=[]
        def addChild(self,element):
            self.children.append(element)
        def getAttribute(self,key):
            return self.children.get(key)
        def getData(self):
            return self.cdata
        def getElements(self,name=''):
            if name:
                return [c for c in self.children if c.name ==name]
            else:
                return list(self.children)
    class Xml2Obj(object):
        '''xml到对象的转换器'''
        def __init__(self):
            self.root = None
            self.nodeStack=[]
        def StartElement(self,name,attributes):
            'Expat start element event handler'
            #实例化一个Element对象
            element=Element(name.encode(),attributes)
            #将元素压入栈中并使之成为子元素
            if self.nodeStack:
                parent=self.nodeStack[-1]
                parent.addChild(element)
            else:
                self.root=element
            self.nodeStack.append(element)
        def EndElement(self,name):
            'Expat end element event handler'
            self.nodeStack[-1].pop()
        def CharacterData(self,data):
            'Expat character data event handler'
            if data.strip():
                data=data.encode()
                element=self.nodeStack[-1]
                element.cdata += data
        def Parse(self,filename):
            #创建Expat解析器
            Parser = expat.ParserCreate()
            #set the expat event handler 
            Parser.StartElementHandler = self.StartElement
            Parse.EndElementHandler = self.EndElement
            Parse.CharacterDataHandler=self.CharacterData
            #解析XML文件
            ParserStatus=Parser.parse(open(filename).read(),1)
            return self.root
    parser=Xml2Obj()
    root_element=parser.Parse('sample.xml')


### 从XML DOM节点的子树中删除仅有空白符的文本节点

    def remove_whilespace_nodes(node):
        """删除DOM节点所有只含空白符的文本后裔"""
        remove_list=[]
        for child in node.childNodes:
            if child.noteType == dom.Node.TEXT_NODE and not child.data.strip():
                #将此文本节点加入到需要删除的节点列表
                remove_list.append(child)
            elif child.hasChildNodes():
                #递归，这是最简单处理子树的方式
                remove_whilespace_nodes(child)
        #执行删除
        for node in remove_list:
            node.parentNode.removeChild(node)
            node.unlink()

### 解析Microsoft Excel 的XML

    import sys
    from xml.sax import saxutils,parse
    class ExcelHandler(saxutils.DefaultHandler):
        def __init__(self):
            self.chars=[]
            self.cells=[]
            self.rows=[]
            self.tables=[]
        def characters(self,content):
            self.chars.append(content)
        def startElement(self,name,atts):
            if name=='Cell':
                self.chars=[]
            elif name=='Row':
                self.cells=[]
            elif name="Table":
                self.rows=[]
        def endElement(self,name):
            if name=='Cell':
                self.cells.append(''.join(self.chars))
            elif name='Row':
                self.rows.append(self.cells)
            elif name='Table':
                self.tables.append(self.rows)
    if __name__ == '__main__':
        excelHandler=ExcelHandler()
        parser(sys.argv[1],excelHandler)
        print excelHandler.tables


### 验证XML文档

    from xml.parsers.xmlproc import utils,xmlval,xmldtd
    def validate_xml_file(xml_filename,app=None,dtd_filename=None):
        #创建验证解析器对象，并带有正确的错误处理者
        parser=xmlval.Validator()
        parser.set_error_handler(utils.ErrorPrinter(parser))
        if dtd_filename is not None:
            #制定dtd文件，载入并设置DTD
            dtd=xmldtd.load_dtd(dtd_filename)
            parser.val.dtd=parser.dtd=parser.ent=dtd
        if app is not None:
            #请求处理应用，设置应用对象
            parser.set_application(app)
        #各种设置完成，最后进行解析
        parser.parse_resource(xml_filename)
        '''如果XML数据是一个字符串，而不是一个文件，调用下面的方法
        parser.feed(s)
        parser.clost()
        '''

### 过滤属于指定命名空间的元素和属性

    from xml impoer sax
    from xml.sax import handler,saxutils,xmlreader
    #我们想要过滤的命名空间
    RDF_NS="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
    clss RDFFilter(saxutils.XMLFilterBase):
        def __init__(self,*args):
            saxutils.XMLFilterBase.__init__(self,*args)
            self.in_rdf_stack=[False]
        def startElementNS(self,(uri,localname),qname,attrs):
            if uri == RDF_NS or self.in_rdf_stack[-1]==True:
                #如果命名空间是RDF或这该元素嵌套在RDF元素中
                #则跳过该元素 ---栈增长
                self.in_rdf_stack.append(True)
                return
            #制作一个不属于RDF命名空间的属性字典
            keep_attrs={}
            for key,value in attrs.items():
                uri,localname =key
                if url != RDF_NS:
                    uri,localname=key
                    if uri != RDF_NS:
                        keep_attrs[key) =value
                #准备已经清理过的非RDF命名空间的属性
                attrs=xmlreader.AttributesNSImpl(keep_attrs,attrs.getQNamers())
                #通过复制最后一个条目来增长栈
                self.in_rdf_stack.append(self.in_rdf_stack[-1])
                #最后将余下的操作委托给基类
                saxutils.XMLFilterBase.startElementNS(self,(uri,localname),qname,attrs)
        def characters(self,content):
            #跳过在RDF命名空间的标签中的字元
            if self.in_rdf_stack[-1]:
                return
            #将余下的操作委托给基类
            saxutils.XMLFilterBase.characters(self,content)
        def endElementNS(self,(uri,localname),qname):
            #弹出栈---什么也不用做，以为我们自是跳过
            if self.in_rdf_stack.pop() == True:
                pass
            #将余下的操作委托给基类
            saxutils.XMLFilterBase.endElementNS(self,(uri,localname),qname)
        def filter_rdf(input,output):
            '''filter_rdf(input=some_input_file,output=some_output_file)
            解析来自输入流的XML输入，过滤掉所有属于RDF命名空间的元素和属性
            '''
            output_gen=saxutil.XMLGenerator(output)
            parser=sax.make_parser()
            filter=RDFFilter(parser)
            filter.setFeature(handler.feature_namespaces,True)
            filter.setContentHandler(output_gen)
            filter.setErrorHandler(handler.ErrorHandler())
            filter.parse(input)
        if  __name__ == '__main__':
            import StringIO,sys
            TEST_RDF='''<?xml version="1.0"?>
            <metadata xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
            xmlns:dc="http://purl.org/dc/elements/1.1/">
            <title>This is non-RDF content</title>
            <rdf:RDF>
                <rdf:Description rdf:about="%s">
                    <dc:Creator>%s</dc:Creator>
                </rdf:Description>
            </rdf:RDF>
            <element/>
            </metadata>
            '''
            input=StringIO.StringIO(TEST_RDF)
            filter_rdf(input,sys.stdout)
