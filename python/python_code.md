# python code

~~~py
from decimal import Decimal
from fun_two_sum import sum1
import requests
import pymysql
import logging

logger = logging.getLogger()


# 这个是第一个注释
print("hello world")

print(3 + 1)

fp = open("E:/a.txt", 'a+')
print("hello world\nhello world1", file=fp)
fp.close()

a = 1.1
b = 2.2
print(a + b)

print(Decimal(a) + Decimal(b))

# 整除的结果
print(10 // 3)
# 一个数字的幂次方
print(2 ** 3)

# 循环求和操作
result = 0
a = 1
while a <= 100:
    result += a
    a += 1
print("这里会得到1-100数字的和为：%d" % result)

# 循环求和操作
result = 0
a = 1
while a <= 100:
    if a % 2 != 0:
        a += 1
        continue
    else:
        result += a
        a += 1
print("这里会得到1-100数字中偶数的和为：%d" % result)
print("你的名字为：%s, 你的年龄为：%d, 身高为：%f" % ("赵小雄", 20, 1.55))

print(sum1(10, 10))

# 列表操作
list_1 = {"zhao", "xiao", "xiong"}
print(len(list_1))
list_1.add("shi")
print(list_1.remove("xiao"))
print(list_1)

# http请求操作
result = requests.get("https://baidu.com")
print(result.status_code)
print(result.headers)
print(result.content)
result.close()

# 数据库操作
db = pymysql.connect(host="localhost", user="root", password="123456", database="test")
cursor = db.cursor()

# 查询操作
id = 0
try:
    cursor.execute(query="select id from t_user")
    data = cursor.fetchall()
    for line in data:
        num = line[len(line)-1]
        if id < num:
            id = num
except Exception as e:
    print("查询执行操作出现错误! %s" % e)

print("当前数据库最大的id值为 %d" % id)

# 新增数据操作
id += 1
try:
    sql = "insert into t_user values ({}, 'xiaoxiong')"
    sql = sql.format(id)
    cursor.execute(query=sql)
except Exception as e:
    print("插入数据失败！%s" % e)

# 更新操作
try:
    sql = "update t_user set name = 'zhang_miao' where id = {}"
    sql = sql.format(id)
    cursor.execute(sql)
except Exception as e:
    print("更新数据失败！ %s" % e)

# 删除操作
try:
    sql = "delete from t_user where id = {}"
    sql = sql.format(10)
    cursor.execute(sql)
except Exception as e:
    logger.error("数据库删除操作执行失败！{}", e)


# 事务提交
db.commit()
# 资源关闭
cursor.close()
db.close()

# shell操作
~~~

