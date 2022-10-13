#### oracle数据生成数据文件

```shell
#!/bin/bash
. ./.bash_profile
filetime=$(date "+%Y%m%d%H%M%S")
##
txtFilePath=/home/saleCenter/dbfile/3.0/bictitem_income/txt/bictitem_income_info_month.txt
oldFile=/home/saleCenter/dbfile/3.0/bictitem_income/oldFile/bictitem_income_info_month_${filetime}.txt

logFilePath=/home/saleCenter/dbfile/3.0/bictitem_income/log/bictitem_income_info_month.log
oldLog=/home/saleCenter/dbfile/3.0/bictitem_income/oldLog/bictitem_income_info_month_${filetime}.log


dbusr='hbdx_sale_db'
dbpwd='sitech_0901'
dbsid='133.0.186.156:1521/sitech_sale'
# $user_name/$password@hostname:$port/$service_name

#连接数据库，<<表示间隔符，在两个！中间写东西，
sqlplus ${dbusr}/${dbpwd}@${dbsid} <<!
delete  from  BICTITEM_INCOME_MONTH;
commit;
insert into BICTITEM_INCOME_MONTH 
select * from (
	select t.oncycledetailid,t.itemid,t.contractid,t.contractcode,contracttype,
	(select min(std_acct_type_id) from cde_jt_acct_type where acct_code=t.inputitem_on and sl=t.taxrate_on) std_acct_type_id,t.inputitem_on,PRO01,PRO02,openflag,taxation,fptype_on,taxrate_on,bustype_on,accounttype_on,t.confirmfee_on,t.begindate_on,'周期型' cycle_name, remark5,t.instant_id,t.create_date,flag,wf_id,wf_state, to_char(sysdate,'yyyymm') wr_date,substr(keyvalcode,instr(keyvalcode,'_',-1)+1) subkeyvalcode,0 cycle_flag 
	from BICTITEM_ONCYCLE_DETAIL_T t 
	where  t.instant_id=(select max(instant_id) from BICTITEM_ONCYCLE_DETAIL_T where contractcode=t.contractcode) 
		and t.instant_id=(select max(instant_id) from bictitem_oncycle_t where itemid=t.itemid) 
		and t.oncycleid in (select oncycleid from bictitem_oncycle_t ) 
union all
	select t.offcycleid,t.itemid,t.contractid,t.contractcode,contracttype,(select min(std_acct_type_id) from cde_jt_acct_type where acct_code=t.inputitem_off and sl=t.taxrate_off) std_acct_type_id, t.inputitem_off,PRO01,PRO02,openflag,taxation,fptype_off,taxrate_off,bustype_off,accounttype_off,t.planconfirmfee_off,t.planconfirmdate_off,'非周期型' cycle_name, remark5,t.instant_id,t.create_date,flag,wf_id,wf_state, to_char(sysdate,'yyyymm')  wr_date,substr(keyvalcode,instr(keyvalcode,'_',-1)+1) subkeyvalcode,1 cycle_flag 
	from BICTITEM_OFFCYCLE_T t 
	where instant_id =(select max(instant_id) from BICTITEM_OFFCYCLE_T where contractcode=t.contractcode)) tt
where tt.openflag=0 and to_char(tt.begindate_on,'yyyymm')= to_char(sysdate,'yyyymm');
COMMIT;

delete from BICTITEM_INCOME_INFO_MONTH;
delete from BICTITEM_INCOME_INFO  where wr_date=to_char(SYSDATE,'yyyymm');

commit;
insert into BICTITEM_INCOME_INFO_MONTH
(
	select tt.oncycledetailid,tt.itemid,tt.contractid,tt.contractcode,q.concode3class,tt.std_acct_type_id,tt.inputitem_on,tt.pro01,tt.pro02,tt.openflag,tt.taxation,
tt.fptype_on,tt.taxrate_on,tt.bustype_on,tt.accounttype_on,tt.confirmfee_on,tt.begindate_on,tt.cycle_name,tt.remark5,tt.instant_id,tt.create_date,tt.flag,tt.wf_id,tt.wf_state,
tt.ctacode,q.contractname,tt.itemcategory,tt.profitcenter,tt.companycode, tt.itemcode,tt.itemname,tt.billno,tt.costcenter,q.tradesum,q.signenddate,q.signdate,q.enddate, to_char(sysdate,'yyyymm') wr_date,tt.subkeyvalcode,tt.cycle_flag 
	from (
		select t.oncycledetailid,t.itemid,t.contractid,t.contractcode,t.contracttype,t.std_acct_type_id,t.inputitem_on,t.pro01,t.pro02,t.openflag,t.taxation, t.fptype_on,t.taxrate_on,t.bustype_on,t.accounttype_on,t.confirmfee_on,t.begindate_on,t.cycle_name,t.remark5,t.instant_id,t.create_date,t.flag,t.wf_id,t.wf_state, n.contractcode ctacode,n.itemcategory,n.profitcenter,n.companycode,n.itemcode,n.itemname,n.billno,n.costcenter,t.subkeyvalcode,t.cycle_flag
		from (select * from BICTITEM_INCOME_MONTH where wr_date =to_char(sysdate,'yyyymm') ) t
		left join (select * from jt_ict_item_t m where nvl(instant_id,0) =(select max(nvl(instant_id,0)) from jt_ict_item_t where itemid=m.itemid)) n
		on t.itemid=n.itemid
	) tt
	left join (
		select * from jt_ict_contract_info_t p where concode3class not in ('A-8') and nvl(instant_id,0) =(select max(nvl(instant_id,0)) from jt_ict_contract_info_t where 
contractcode=p.contractcode))  q
	on tt.ctacode=q.contractcode
group by tt.oncycledetailid,tt.itemid,tt.contractid,tt.contractcode,q.concode3class,tt.std_acct_type_id,tt.inputitem_on,tt.pro01,tt.pro02,tt.openflag,tt.taxation, tt.fptype_on,tt.taxrate_on,tt.bustype_on,tt.accounttype_on,tt.confirmfee_on,tt.begindate_on,tt.cycle_name,tt.remark5,tt.instant_id,tt.create_date,tt.flag,tt.wf_id,tt.wf_state,
tt.ctacode,q.contractname,tt.itemcategory,tt.profitcenter,tt.companycode,tt.itemcode,tt.itemname,tt.billno,tt.costcenter,q.tradesum,q.signenddate,q.signdate,q.enddate,tt.subkeyvalcode,tt.cycle_flag);
COMMIT;
insert into BICTITEM_INCOME_INFO select *  from  BICTITEM_INCOME_INFO_MONTH;
COMMIT;
exit
!


sqlplus ${dbusr}/${dbpwd}@${dbsid} <<!
set echo off;
set feedback off;
set tab off;
set heading off;
set pagesize 0;
set linesize 3000; 
set numwidth 12;
set termout off;
set trimout on;
set trimspool on;
spool $txtFilePath;
select oncycledetailid||'||'||itemid||'||'||contractid||'||'||contractcode||'||'||contracttype||'||'||std_acct_type_id||'||'||inputitem_on||'||'||pro01||'||'||pro02||'||'||openflag||'||'||taxation||'||'||fptype_on||'||'||taxrate_on||'||'||bustype_on||'||'||accounttype_on||'||'||confirmfee_on||'||'||TO_CHAR(begindate_on,'yyyymmddhh24miss')||'||'||cycle_name||'||'||remark5||'||'||instant_id||'||'||TO_CHAR(create_date,'yyyymmddhh24miss')||'||'||flag||'||'||wf_id||'||'||wf_state||'||'||ctacode||'||'||contractname||'||'||itemcategory||'||'||profitcenter||'||'||companycode||'||'||itemcode||'||'||itemname||'||'||billno||'||'||costcenter||'||'||tradesum||'||'||signenddate||'||'||signdate||'||'||enddate||'||'||wr_date||'||'||subkeyvalcode||'||'||cycle_flag  from bictitem_income_info_month;
spool off;
exit
!

sleep 3
python  /home/saleCenter/dbfile/3.0/bictitem_income/bictitem_income_info_month.py
sleep 3
mv $txtFilePath  $oldFile
mv $logFilePath  $oldLog

echo   $txtFilePath
```

通过生成文件去插入到目标库

```Python
# -*- coding: UTF-8 -*-
#!/bin/env python
import MySQLdb
import time
import sys
import os
import logging


print time.strftime('%Y%m%d %H:%M:%S')
stdout_backup = sys.stdout
log_file = open("/home/saleCenter/dbfile/3.0/bictitem_income/log/bictitem_income_info_month.log", "w")
sys.stdout = log_file
t1 = time.time()
print "begin time",time.strftime('%Y-%m-%d %H:%M:%S')
filename = "/home/saleCenter/dbfile/3.0/bictitem_income/txt/bictitem_income_info_month.txt"
print "filename ",filename
conn = MySQLdb.connect(host='133.0.205.199',port=8918,user='dbsaleadm',passwd='QeyW828_d8',db='SALECENTER',charset='UTF8',)
cur = conn.cursor()

def main():
    try:
        cur.execute( "delete from bictitem_income_info where wr_date=DATE_FORMAT(SYSDATE(),'%Y%m')" )
        conn.commit()
        print "delete bictitem_income_info by month"
    except Exception as e:
        print e
        print "Exception:"
        conn.rollback()
    countStr = 0
    failcount = 0
    totalcount = 0
    param=[]
    for line in file(filename):
        line = line.strip()
        if line.find("SQL")!=-1:
            print "SQL:",line
            continue
        lineStr = line.split("||")
        if len(lineStr) < 40:
            failcount += 1
            print "err line:",failcount
            continue
        ##字段不能少 state_date = datetime.datetime.strptime(lineStr[1],"%Y%m%d%H%M%S")
        std_acct_type_id = lineStr[5]
        if len(lineStr[5]) == 0:
            std_acct_type_id = None
        instant_id = lineStr[19]
        if len(lineStr[19]) == 0:
            instant_id = None
        create_date = lineStr[20]
        if len(lineStr[20]) == 0:
            create_date = None
        wf_id = lineStr[22]
        if len(lineStr[22]) == 0:
            wf_id = None
        tradesum = lineStr[33]
        if len(lineStr[33]) == 0:
            tradesum = 0
        cycle_flag = lineStr[39]
        if len(lineStr[39]) == 0:
            cycle_flag = None
        param.append([lineStr[0],lineStr[1],lineStr[2],lineStr[3],lineStr[4],std_acct_type_id,lineStr[6],lineStr[7],lineStr[8],lineStr[9],lineStr[10],lineStr[11],lineStr[12],lineStr[13],lineStr[14],lineStr[15],lineStr[16],lineStr[17],lineStr[18],instant_id,create_date,lineStr[21],wf_id,lineStr[23],lineStr[24],lineStr[25],lineStr[26],lineStr[27],lineStr[28],lineStr[29],lineStr[30],lineStr[31],lineStr[32],tradesum,lineStr[34],lineStr[35],lineStr[36],lineStr[37],lineStr[38],cycle_flag])
        if countStr >= 1000:
            if param is not None:
                try:  
                    sql = 'insert into bictitem_income_info(oncycledetailid,itemid,contractid,contractcode,contracttype,std_acct_type_id,inputitem_on,pro01,pro02,openflag,taxation,fptype_on,taxrate_on,bustype_on,accounttype_on,confirmfee_on,begindate_on,cycle_name,remark5,instant_id,create_date,flag,wf_id,wf_state,ctacode,contractname,itemcategory,profitcenter,companycode,itemcode,itemname,billno,costcenter,tradesum,signenddate,signdate,enddate,wr_date,subkeyvalcode,cycle_flag) values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)'
                    cur.executemany(sql, param)
                    conn.commit()
                except Exception as e:
                    failcount += 1
                    print "insert:", line, e
                    conn.rollback()
                    continue
            countStr = 0
            param[:]=[]
        totalcount += 1
        countStr += 1
    if param is not None:
        try:  
            sql = 'insert into bictitem_income_info(oncycledetailid,itemid,contractid,contractcode,contracttype,std_acct_type_id,inputitem_on,pro01,pro02,openflag,taxation,fptype_on,taxrate_on,bustype_on,accounttype_on,confirmfee_on,begindate_on,cycle_name,remark5,instant_id,create_date,flag,wf_id,wf_state,ctacode,contractname,itemcategory,profitcenter,companycode,itemcode,itemname,billno,costcenter,tradesum,signenddate,signdate,enddate,wr_date,subkeyvalcode,cycle_flag) values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)'
            cur.executemany(sql, param)
            conn.commit()
        except Exception as e:
            failcount += 1
            print "insertparam:", line, e
            conn.rollback()
        countStr = 0
        param[:]=[]
    t2 = time.time()
    cur.close()
    conn.close()
    t2 = time.time()
    print "t2-t1", t2-t1
    print "totalcount:", totalcount
    print "failcount:", failcount
    print "end time",time.strftime('%Y-%m-%d %H:%M:%S')
main()
log_file.close()
sys.stdout = stdout_backup
print time.strftime('%Y%m%d %H:%M:%S')
```











#### mysql本库中的数据处理

```python
# -*- coding: UTF-8 -*-
#!/bin/env python

import MySQLdb
import time
import sys
import os
import logging


timeStr=time.strftime('%Y%m')
tableName='bictitem_income_info_'+timeStr
if len(sys.argv) > 1:
    if len(sys.argv[1]) == 6:
        tableName = 'bictitem_income_info_'+sys.argv[1]
print tableName
print time.strftime('%Y%m%d %H:%M:%S')
stdout_backup = sys.stdout
log_file = open("/home/saleCenter/dbfile/3.0/mysql_to_mysql/log/bictitem_income_info.log", "w")
sys.stdout = log_file
t1 = time.time()
print "begin time",time.strftime('%Y-%m-%d %H:%M:%S')
conn = MySQLdb.connect(host='133.0.205.199',port=8918,user='dbsaleadm',passwd='QeyW828_d8',db='SALECENTER',charset='UTF8',)
cur = conn.cursor()

def main():
    try:
        cur.execute( "delete from "+tableName )
        conn.commit()
        print "delete tableName by month"
    except Exception as e:
        print e
        print "Exception:"
        conn.rollback()
    countStr = 0
    failcount = 0
    totalcount = 0
    preInfoSql="select oncycledetailid,itemid,contractid,contractcode,contracttype,std_acct_type_id,inputitem_on,pro01,pro02,openflag,taxation,fptype_on,taxrate_on,bustype_on,accounttype_on,confirmfee_on,begindate_on,cycle_name,remark5,instant_id,create_date,flag,wf_id,wf_state,ctacode,contractname,itemcategory,profitcenter,companycode,itemcode,itemname,billno,costcenter,tradesum,signenddate,signdate,enddate,wr_date,subkeyvalcode,cycle_flag from bictitem_income_info limit 200000"
    cur.execute(preInfoSql)
    preInfoResult = cur.fetchall()
    param=[]
    for lineStr in preInfoResult:
        param.append([lineStr[0],lineStr[1],lineStr[2],lineStr[3],lineStr[4],lineStr[5],lineStr[6],lineStr[7],lineStr[8],lineStr[9],lineStr[10] ,lineStr[11],lineStr[12],lineStr[13],lineStr[14],lineStr[15],lineStr[16],lineStr[17],lineStr[18],lineStr[19],lineStr[20] ,lineStr[21],lineStr[22],lineStr[23],lineStr[24],lineStr[25],lineStr[26],lineStr[27],lineStr[28],lineStr[29],lineStr[30] ,lineStr[31],lineStr[32],lineStr[33],lineStr[34],lineStr[35],lineStr[36],lineStr[37],lineStr[38],lineStr[39]])
        if countStr >= 2000:
            if param is not None:
                try:  
                    sql= 'insert into '+tableName+ ' (oncycledetailid,itemid,contractid,contractcode,contracttype,std_acct_type_id,inputitem_on,pro01,pro02,openflag,taxation,fptype_on,taxrate_on,bustype_on,accounttype_on,confirmfee_on,begindate_on,cycle_name,remark5,instant_id,create_date,flag,wf_id,wf_state,ctacode,contractname,itemcategory,profitcenter,companycode,itemcode,itemname,billno,costcenter,tradesum,signenddate,signdate,enddate,wr_date,subkeyvalcode,cycle_flag) values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)'
                    cur.executemany(sql, param)
                    conn.commit()
                except Exception, e:
                    failcount += 1
                    print "insert:", e
                    conn.rollback()
                    continue
            countStr = 0
            param[:]=[]
        totalcount += 1
        countStr += 1
    if param is not None:
        try:  
            sql= 'insert into '+tableName+ ' (oncycledetailid,itemid,contractid,contractcode,contracttype,std_acct_type_id,inputitem_on,pro01,pro02,openflag,taxation,fptype_on,taxrate_on,bustype_on,accounttype_on,confirmfee_on,begindate_on,cycle_name,remark5,instant_id,create_date,flag,wf_id,wf_state,ctacode,contractname,itemcategory,profitcenter,companycode,itemcode,itemname,billno,costcenter,tradesum,signenddate,signdate,enddate,wr_date,subkeyvalcode,cycle_flag) values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)'
            cur.executemany(sql, param)
            conn.commit()
        except Exception, e:
            failcount += 1
            print "insertparam:",  e
            conn.rollback()
        countStr = 0
        param[:]=[]
    t2 = time.time()
    cur.close()
    conn.close()
    t2 = time.time()
    print "t2-t1", t2-t1
    print "totalcount:", totalcount
    print "failcount:", failcount
    print "end time",time.strftime('%Y-%m-%d %H:%M:%S')
main()
log_file.close()
sys.stdout = stdout_backup
print time.strftime('%Y%m%d %H:%M:%S')
```

