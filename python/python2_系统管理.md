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
                randspot=random.randrange(len(self.data))
                self.data=self.data[randspot:]+self.data[:randspot]
                #获得n-gram
                where=-1
                #试图定位self.data中maxmem个字符
                #如果i<maxmem,我们其实只获取最后i个
                #即使是所有self.chars也没有问题
                locate=''.join(self.chars[-maxmem:])
                while where < 0 and locate:
                    #定位data中的n-gram
                    where=self.data.find(locate)
                    #如果有必要的话后退到一个短一点的n-gram
                    locate=locate[1:]
                #如果where=-1 且locate='' , 选择self.data[0]
                #那是self.data中随机的一项，因为旋转过了
                c=self.data[where+len(locale)+1]
                #我们只需要小写字母，所以，如果我们挑到了
                #大写字母，我们会随机再次选择一个字母
                if not c.islower():
                    c=random.choice(string.lowercase)
                #最后我们将字母记录到self.chars
                self.chars.append(c)
        def __str__(self):
            return ''.join(self.chars)
    if __name__=='__main__':
        import sys
        if len(sys.argv) > 1: dopass=int(sys.argv[1])
        else:
            dopass = 8
        if len(sys.argv)>2 :
            length = int(sys.argv[2])
        else:
            length = 10
        if len(sys.argv) >3 :
            memory=int(sys.argv[3])
        else:
            memory=3
        onepass=password()
        for i in range(dopass):
            onepass.renew(length,memory)
            print oenpass

### 以pop服务器的方式验证用户

    def popAuth(popHost,user,passwd):
        import poplib
        try:
            pop=poplib.POP3(popHost)
        except:
            raise RuntimeError("Could not establist connection to %r \
            for password check" % popHost)
        try:
            pop.user(user)
            pop.pass_(passwd)
            length,size=pop.stat()
            assert type(length) == type(size) == int
        except:
            raise RuntimeError("Could not verify identity.\n \
            User name %r or password incorrect." % user)
        finally:
            pop.quit()


### 统计Apache中每个IP的点击率

    import re
    def calculateApacheIpHits(logfile_pathname):
        '''返回一个及将IP地址映射到点击率的字典'''
        ipHitListing={}
        ip_spec=r'\.'.join([r'\d{1,3}']*4)
        re_ip=re.compile(ip_spec)
        contents=open(logfile_pathname,r)
        #处理文件中的每一行
        for line in contents:
            match=re_ip.match(line)
            if match：
                ip=match.group()
                #确保IP地址是正确的
                try:
                    quad=map(int,ip.split('.'))
                except ValueError:
                    pass
                else:
                    if len(quad) == 4 and min(quad) >=0  and max(quad)<=255:
                        ipHitListing[ip]=ipHitListing.get(ip,0)+1
        return ipHitListing

### 在脚本中调用编辑器

    import os,sys,tempfile
    def what_editor():
        editor=os.getenv('VISUAL') or os.getenv('EDITOR')
        if not editor:
            if sys.platform=='windows':
                editor ='Notepad.EXE'
            else:
                editor = 'vi'
        return editor
    def edited_text(starting_text=''):
        temp_fd,temp_filename = tempfile.mkstemp(text=True)
        os.write(temp_fd,starting_text)
        os.close(temp_fd)
        editor = what_editor()
        x=os.spawnlp(os.P_WAIT,editor,editor,temp_filename)
        if x:
            raise RuntimeError("Can't run %s %s (%s)" % (editor,temp_filename,x))
        result = open(temp_filename).read()
        os.unlink(temp_filename)
        return = result
    if __name__=='__main__':
        text=edited_text('''Edit this text a little,go ahead,
            it's just a demonstration,after all...!''')
        print 'Edited text is:' ,text


### 备份文件

    import sys,os.shutil,filecmp
    MAXVERSION=100
    def backup(tree_top,backdir_name="bakdir"):
        for dir,subdirs,files in os.walk(tree_top):
            #确保每个目录都有个子目录叫bakdir
            backup_dir=os.path.join(dir,bakdir_name)
            if not os.path.exists(backup_dir):
                os.makedirs(backup_dir)
            #停止对备份目录的递归
            subdirs[:] =[d for d in subdirs if d != bakdir_name]
            for file in files:
                filepath=os.path.join(dir,file):
                destpath=os.path.join(backup_dir,file)
                #检查以前的版本的存在
                for index in xrange(MAXVERSION):
                    backup='%s.%2.2d' % (destpath,index):
                    if not os.path.exists(backup):beak

            if index > 0:
                #如果文件和最新版本的文件相同，不需要备份
                old_backup='%s.2.2%d' % (destpath,index-1)
                abspath=os.path.abspath(filepath)
                try:
                    if os.path.isfile(old_backup) and filecmp.cmp(abspath,old_backup,shallow=False):
                        continue
                except OSError:
                    pass
                try:
                    shutil.copy(filepath,backup)
                except OSError:
                    pass
