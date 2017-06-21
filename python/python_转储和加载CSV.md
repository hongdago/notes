python_转储和加载CSV.md
=============================
### 将简单的序列转储为CSV
创建一个CSV writer需要一下3个步骤
1. 以newline选项(赋值为"")打开一个文件，这是为了支持(可能)CSV文件中非标准的行
2. 创建一个CSV writer 对象
3. 在文件的第一行设定标题。使用writer对象中的writerow()方法写入csv文件
>示例

    import string
    import random
    from collections import namedtuple

    Person=namedtuple("Person","key,name,gender")

    def as_dict(person):
        return dict(key=person.key,name=person.name,gender=person.gender)

    def person_iter(max=30):
        for i in range(30):
            yield Person("{0:04d}".format(i),
                "".join([random.choice(string.ascii_letters) for i in range(5)]),
                random.choice(["male","female"]))
    import csv
    with open("person.csv","w",newline="") as target:
        writer=csv.DictWriter(target,Person._fields)
        writer.writeheader()
        for person in person_iter():
            writer.wirterow(as_dict(person))

### 从CSV文件中加载简单的序列

    import csv
    with open("person.csv","r",newline="") as source:
        reader=csv.DictReader(source)
        assert set(reader.fieldnames) == set(Person._fields)  #校验文件格式
        for person in (Person(**r) for r in reader):
            print(person)

