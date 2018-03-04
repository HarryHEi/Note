---
title: 图片插入mysql
date: 2016-03-12 10:17:24
tags: [python,mysql]
---


# 功能
此程序把一个文件夹所有图片存入mysql

# 存入图片
```
#!/usr/bin/env python
# -*-coding:utf-8 -*-
"此程序用于将一个文件夹下所有图片以JPG格式保存到mysql数据库"

import MySQLdb as mdb
from PIL import Image
import sys,os

url = "/home/herui/图片/"
#存放临时文件
pic2 = "temp.jpg"
count_false = 0
count_all = 0
false_file=[]

try:
    con = mdb.connect("localhost", "flask", "admin", "flaskdb")
    cur= con.cursor()

    #循环读取图片
    for root,dirs,files in os.walk(url):
        for pic in files:

            long_url = os.path.join(root,pic)
            name = str(pic).split('.')[0]
            name_end = str(pic).split('.')[1]
            print long_url
            #检测后缀名
            if name_end in ("jpg","JPG","png","PNG"):
                count_all += 1
            #转换图片格式
                try:
                    Image.open(long_url).save(pic2,"JPEG")
                except:
                    #记录打开失败的文件
                    print pic+"打开失败"
                    false_file.append(long_url)
                    count_false += 1
                    continue

                print "格式转换成功~"

                f = open(pic2)
                img = f.read()
                f.close()

                print "读取成功～"

                cur.execute("insert into images set data = '%s' "% mdb.escape_string(img))

    print "更新数据库,总共%d,失败%d" %(count_all,count_false)

    for false_f in false_file:
        print false_f

    con.commit()

except IOError, e:
    con.rooback()
    print "Error %d %s" %(e.args[0], e.args[1])
    sys.exit(1)

cur.close()
con.close()
```

# 读取图片
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

"读取图片"

import MySQLdb as mdb
import sys

try:
    con = mdb.connect("localhost", "flask", "admin", "flaskdb")
    cur = con.cursor()

    cur.execute("select data from images")

#返回行数
    numrows = int(cur.rowcount)
#循环读取图片
    for row in range(numrows):
#文件名 
        pic = "image"+str(row)+".jpg"
#写入文件
        f = open(pic, "wb")
        f.write(cur.fetchone()[0])
        f.close()

        print pic+"OK"

except IOError, e:
    con.rollback()
    print "Error %d %s" %(e.args[0], e.args[1])
    sys.exit(1)

cur.close()
con.close()
```