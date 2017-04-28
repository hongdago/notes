系统管理
=====================
### 生产随机密码

    from random import choice
    import string
    def GenPasswd(length=9,chars=string.letters+string.digits):
        return ''.join([choice(char) for i in range(8)])

### 生成易记的伪随机密码

    import random,string
    class password(object):
        #任意含有大量单词的文件都可以：我们只想让self.data
        #成为一个大字符串
        data=open('/usr/share/dict/words').read().lower()
        def renew(self,n,maxmem=3):
            '''根据回溯的最大“历史”的maxmen个字符
            在self.chars中积累n个随机字符'''
            self.chars=[]
            for i in range(n):
                #随机“旋转”self.data
                randspot=random.
