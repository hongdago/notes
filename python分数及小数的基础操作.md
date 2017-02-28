## python分数及小数的基础操作
### 小数数字
>从功能来说，小数对象就像浮点数，只不过它们有固定的位数和小数点，因此**小数是有固定的精度的浮点数**

    >>> 0.1+0.1+0.1 -0.3
    5.551151231257827e-17
    >>> print(0.1+0.1+0.1-0.3)
    5.551151231257827e-17
    >>> from decimal import Decimal
    >>> Decimal('0.1')+Decimal('0.1')+Decimal('0.1)-Decimal('0.3')
    Decimal('0.0')

#### 设置全局精度
    
    >>> import decimal
    >>> decimal.Decimal(1) / decimal.Decimal(7)
    Decimal('0.1428571428571428571428571429')

    >>> decimal.getcontext().prec = 4
    >>> decimal.Decimal(1) / decimal.Decimal(7)
    Decimal('0.1429')

### 分数
>分数以类似于小数的方式使用，它也存在与模块中;导入其构造函数并传递一个分子和一个分母就可以产生一个分数


    >>> from fractions import Fraction
    >>> x = Fraction(1,3)
    >>> y = Fraction(4,6)
    >>> x
    Fraction(1,3)
    >>> y
    Fraction(2,3)
    >>> print(y)
    2/3
    >>> x + y
    Fraction(1,1)
    >>> x - y
    Fraction(-1,3)
    >>> x * y
    Fraction(2,9)

#### 转换

    >>>(2.5).as_integer_ratio()
    (5,2)
    >>> f = 2.5
    >>> z = Fraction(*f.as_integer_ratio())
    Fraction(5,2)

    >>> x = Fraction(1,3)
    >>> x + z
    Fraction(17,6)

    >>> float(x)
    0.333333333333331
    >>> float(z)
    2.5
    >>> float(x+z)
    2.833333333333335
    >>> 17/6
    2.833333333333335

    >>> Fraction.from_float(1.75)
    Fraction(7,4)
    >>> Fraction(*(1.75).as_integer_ratio())
    Fraction(7,4）

#### 混合类型

    >>> x
    Fraction(1,3)
    >>> x + 2
    Fraction(7,3)                    #Fraction + int  -> Fraction
    >>> x + 2.0
    2.33333333333335                 #Fraction + flaot -> float
    >>> x +  (1./3 )
    0.66666666666663                 #Fraction + float -> float
    >>> x + Fraction(4,3) 
    Fraction(5,3)                    #Fraction + Fraction -> Fraction

    >>> 4.0 / 3
    1.333333333333333
    >>> (4.0 / 3).as_integer_ratio()
    (6004799503160661,4503599627370496)
    >>> x
    Fraction(1,3)
    >>> a = x + Fraction(*(4.0/3).as_integer_ratio())
    >>> a
    Fraction(22517998136852479,13510798882111488)

    >>> 22517998136852479 / 13510798882111488
    1.66666666666667
    >>> a.limit_denominator(10)
    Fraction(5,3)


