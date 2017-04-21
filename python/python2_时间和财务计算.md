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


