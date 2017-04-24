时间和财务计算(python2)
========================
### 检查平台所使用的纪元

    >>> import time
    >>> print time.asctime(time.gmtime(0))
    Thu Jan  1 00:00:00 1970
    >>> time.gmtime(0)
    time.struct_time(tm_year=1970, tm_mon=1, tm_mday=1, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=3, tm_yday=1, tm_isdst=0)
    >>> time.localtime(0)
    time.struct_time(tm_year=1970, tm_mon=1, tm_mday=1, tm_hour=8, tm_min=0, tm_sec=0, tm_wday=3, tm_yday=1, tm_isdst=0)

### 计算昨天和明天的日期

    import datetime
    today = datetime.date.today()
    #yesterday=today.replace(day=today.day-1)
    yesterday = today - datetime.timedelta(days=1)
    #tomorrow=today.replace(day=today.day+1)
    tomorrow = today + datetime.timedalta(days=1)
    print yesterday,today,tomorrow

### 寻找上一个星期五
    
    import datetime,calendar
    lastFriday = datetime.date.today()
    oneday=datetime.timedelta(days=1)
    while lastFriday.weekday() != calendar.FRIDAY:
        lastFriday -= oneday
    print lastFriday.strftime("%A，%d-%b-%Y")

    #去除循环的方法
    import datetime,calendar
    today = datetime.date.today()
    targetDay=calendar.FRIDAY:
    thisDay = today.weekday()
    deltaToTarget=(thisDay-targetDay)% 7
    lastFriday = today - datetime.timedelta(days=deltaToTarget)
    print lastFriday.strftime("%A，%d-%b-%Y")

### 计算歌曲的总播放时间

    import datetime
    def totaltimer(times):
        td=datetime.timedelta(0)     #将总和初始化
        duration=sum([datetime.timedelta(minutes=m,seconds=s) for m,s in times],
            td)
        return duration
    if __name__=='__main__':
        times1=[(2,36),
                (3,35),
                (3,45),
            ]
        times2=[(3,0),
                (5,13),
                (4,12),
                (1,10)
                ]
        assert totaltimer(times1) == datetime.timedelta(0,596)
        assert totaltimer(times2) == datetime.timedelta(0,815)

### 反复执行某个命令
    
    import time,os,sys
    def main(cmd,inc=60):
        while True:
            os.system(cmd)
            time.sleep(inc)
        if __name__=="__main__":
            numargs = len(sys.argv) -1
            if numargs <1 or numargs >2:
                print "usage: " + sys.argv[0] + " command [seconds_delay]"
                sys.exit(1)
            cmd=sys.argv[1]
            if numargs < 2:
                main(cmd)
            else:
                inc=int(sys.argv[2])
                main(cmd,inc)

### 定时执行命令

    import time,os,sys,sched
    schedule=sched.scheduler(time.time,time.sleep)
    def perform_command(cmd,inc):
        schedule.enter(inc,0,perform_command,(cmd,inc)) #re-scheduler
        os.system(cmd)
    def main(cmd,inc=60):
        schedule.enter(0,0,perform_command,(cmd,inc))
        schedule.run()

### 将十进制数用于货币处理

    import decimal
    """计算意大利支票税"""
    def italformat(value,places=2,curr="EUR",sep='.',dp=',',pos='',neg='-',
        overall=10):
        """将十进制value转化为财务格式的字符串
        places:十进制小数点后面的数字的位数
        curr:可选的币种符号（可能为空)
        sep:可选的分组(三个一组)分割符
        dp:小数点指示符
        pos:正数的可选符号:"+",空格或空白
        neg:负数的可选符号:"-","(",空格或空白
        overall:最终结果的可选总长度，若总长度不够，左边货币符号和数字之间会被
        填充符占据
        """
        q=decimal.Decimal((0,(1,),-places)) #2 places --> '0.01'
        sign,digits,exp = value.quantize(q),as_tuple()
        result=[]
        digits=map(str,digits)
        append,next=result.append,digits.pop
        for i in range(places):
            if digits:
                append(next())
            else:
                append('0')
        append(dp)
        i=0
        while digits:
            append(next())
            i +=1
            if i == 3 and digits:
                i=0
                append(sep)
        while len(result) <overall:
            append(' ')
        append(curr)
        if sign:append(neg)
        else:append(pos)
        result.reverse()
        return ''.join(result)
    #获得计算用的小数：
    def getsubtotal(subtin=None):
        if subtin == None:
            subtin = input("Enter the subtotal:")
            subtotal=decimal.Decimal(str(subtin))
            print "\n subtotal:     ",italformat(subtotal)
            return subtotal
    #指定意大利税法函数
    def cnpcalc(subtotal):
        contrib = subtotal*decimal.Decimal('.02')
        print "+ contributo integerativo 2%: ",italformat(contriv,curr='')
        return contrib

### 检查信用卡校验和

    def cardLuhnChecksumIsValid(card_number):
        """通过lunh mod-10校验和算法检查信用卡号"""
        sum =0 
        num_digits = len(card_number)
        oddeven=numargs & 1
        for count in range(num_digits):
            digit = int(card_number[count])
            if not ((count & 1) ^ oddeven):
                digit = digit * 2
            if digit > 9:
                digit = digit -9
            sum = sum + digit
        return (sum % 10 ) == 0
