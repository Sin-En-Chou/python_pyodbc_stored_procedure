# python_pyodbc_stored_procedure

#Example:

```python
import pyodbc
SERVER = '<server_name or IP>'
DATABASE = '<database_name>'
USERNAME = '<user_name>'
PASSWORD = '<user_pass>'
constr = 'DRIVER={SQL Server};SERVER=' + SERVER + ';DATABASE=' + DATABASE + ';UID=' + USERNAME + ';PWD=' + PASSWORD

def method1():
	cnxn = pyodbc.connect(constr)
	curs = cnxn.cursor()
	curs.execute("{call proc_datdr777(20200701,2,'X00012345','','A')}") # call Stored Procedure

	while curs.nextset():
		try:
			rows = curs.fetchall()
			break
		except pyodbc.ProgrammingError:
			continue

	print(rows)	# Output!
	curs.close()
	cnxn.close()

def method2():
	results = []
	cnxn = pyodbc.connect(constr)
	curs = cnxn.cursor()
	# Stored Procedure Call Statement
	sqlExecSP = "set nocount on; exec proc_datdr777 20200701,2,'X00012345','','A'"
	curs.execute(sqlExecSP)

	columns = [column[0] for column in curs.description] # 抓出所有欄位名稱
	# print(columns)
	for row in curs.fetchall():
		print(row)
		x = dict(zip(columns,row))                         # 將抓出來的值與欄位名稱包裝成dict(字典)型態資料
		results.append(x)                                  # 最後把每一筆資料dict裝在list(列表)中
	print("========================")
	print(results)


""" RESULTS """
method1()
method2()
```
***
這時輸出會發現抓出來的值竟然有`'strike_price': Decimal('71')`
那怎麼辦?! 「只好涼拌炒雞蛋囉！」 (誤)

解決辦法:
目前僅能在自己寫的Stored Procedure將此欄位加上CONVERT()函數去做轉換!
```
CONVERT(data_type(length),data_to_be_converted,style)
```
例:
```mysql
DECLARE @price int
SET @price = 71
--SELECT CONVERT(decimal(14,2),avg(strike_price)) ... -- <-- 你的程式
SELECT CONVERT(decimal(14,2),avg(@price))
```
輸出:
`71`

參考網址:
1. https://stackoverflow.com/questions/46566992/how-to-parse-decimalstr-with-python
2. https://www.codeproject.com/Questions/1025056/I-want-to-return-decimal-values-using-stored-proce
