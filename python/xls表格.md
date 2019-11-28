xls概念：

workbook：工作薄。就是一个xls文件。

worksheet：工作表。就是xls文件中的sheet。

column:列。

row：行。

如下是一个demo：

```python 
import xlsxwriter
work_book = xlsxwriter.Workbook('test.xls') #创建一个xls文件
work_sheet = work_book.add_worksheet('test') #创建一个sheet
work_sheet.set_column('A:A',20) #调整列的宽度，如果将column换成row，则是修改行的行高
work_sheet.write('A2','123') #在指定的单元格写数据
work_book.close()
```

**注意：一定要在程序结尾关闭workbook对象，否则将不会生成文件。**

# write方法表


| 方法       | 功能 |
|----------|----------|
|  write_string| 写字符串 |
|write_number|写数据|
|write_blank|写空数据|
|write_formual|写公式|
|write_datetime|写日期|
|write_boolean|写逻辑型|
|write_url|写url|

write方法有三个参数，分别是行，列，数据。

# 插入对象

可以用insert_*的方法插入多种对象（图片，按钮等）。有三个参数，分别是插入位置，插入对象文件位置，url。

# chat类

即是图标类。用add_chat({type:'column'})置图标。

```python
hat = work_book.add_chart({'type':'column'})
chat.add_series({
        'categories':'=test!$A$1:$A$5',
        'values':'=test!$B$1:$B$5',
        'line': {'color':'red'}
    })
work_sheet.insert_chart('D7',chat)
```

categories:横坐标的数据源。
values：统计图的数据源。
line：统计图的边框线。



