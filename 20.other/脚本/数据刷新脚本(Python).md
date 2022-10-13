数据刷新脚本，全量刷新表中数据

```python
# -*- coding: UTF-8 -*-
# !/bin/env python

import MySQLdb
import time
import sys

tableName = 'item_controlType_all'
print tableName
print time.strftime('%Y%m%d %H:%M:%S')
stdout_backup = sys.stdout
log_file = open("/home/saleCenter/dbfile/3.0/mysql_to_mysql/log/province_control_type.log", "w")
sys.stdout = log_file
t1 = time.time()
print "begin time", time.strftime('%Y-%m-%d %H:%M:%S')
conn = MySQLdb.connect(host='133.0.205.199', port=8918, user='dbsaleadm', passwd='QeyW828_d8', db='SALECENTER',
                       charset='UTF8')
cur = conn.cursor()


def main():
    try:
        cur.execute("delete from " + tableName +" where 1=1")
        conn.commit()
    except Exception as e:
        print "Exception:" + e
        conn.rollback()
    countStr = 0
    failcount = 0
    totalcount = 0
    preInfoSql = "select a.itemCode ITEMCODE,a.controltypecode CONTROLTYPECODE,IFNULL(a.provinceControltypecode,a.controltypecode) as PROVINCECONTROLTYPECODE from (SELECT t.itemcode itemCode, t.controltypecode controltypecode, (select province_depart_id from project_control_department where itemCode=t.itemcode) provinceControltypecode FROM bictitem t where t.controltypecode !='' ) as a limit 200000"
    cur.execute(preInfoSql)
    preInfoResult = cur.fetchall()
    param = []
    for lineStr in preInfoResult:
        param.append([lineStr[0], lineStr[1], lineStr[2]])
        if countStr >= 2000:
            if param is not None:
                try:
                    sql = 'insert into ' + tableName + ' (itemcode,controltypecode,provincecontroltypecode) values(%s,%s,%s)'
                    cur.executemany(sql, param)
                    conn.commit()
                except Exception as e:
                    failcount += 1
                    print "insert:", e
                    conn.rollback()
                    continue
            countStr = 0
            param[:] = []
        totalcount += 1
        countStr += 1
    if param is not None:
        try:
            sql = 'insert into ' + tableName + ' (itemcode,controltypecode,provincecontroltypecode) values(%s,%s,%s)'
            cur.executemany(sql, param)
            conn.commit()
        except Exception as e:
            failcount += 1
            print "insertparam:", e
            conn.rollback()
        countStr = 0
        param[:] = []
    cur.close()
    conn.close()
    t2 = time.time()
    print "t2-t1", t2 - t1
    print "totalcount:", totalcount
    print "failcount:", failcount
    print "end time", time.strftime('%Y-%m-%d %H:%M:%S')


main()
log_file.close()
sys.stdout = stdout_backup
print time.strftime('%Y%m%d %H:%M:%S')
```

