## NSE脚本引擎

- 升级： sudo nmap --script-updatedb
- 位置：`/usr/share/nmap/scripts/`

- 文档：https://nmap.org/nsedoc/
- 目前默认包含500多个脚本（14大类）
	- Auth、Broadcast、Brute、Default、Discovery、Dos、Exploit、 External、Fuzzer、intrusive、Malware、Safe、Version、Vuln

---

### 使用NSE脚本

- `-sC`
	default类脚本
- `--script="http-*"、safe、auth、vuln、exploit、http-title`
- `--script-args=`
- `nmap --script="banner, upnp-info, firewalk, http-title" -p 80`
- `nmap --script="not(intrusive or dos or exploit)" -sV`
- `nmap --script="http-* and not(http-slowloris or http-brute or http-enum or http-form-fuzzer)"`

---
### Nmap脚本

- Head
	描述、注释、定义变量、引用模块、规定类型、license
	- `--`单行注释
	- `[[ ]]`多行注释
		- `Description` 描述用途等
	- `author= Tom`
	- `license= "same as Nmap-- See https://nmap.org/book/man-legal.html"`
	- `categories= {"deault", "discovery", "safe"}`
	- `local http= require "http"`		# 引用模块并定义变量
	- `local nmap= require "nmap"`
- Rule
	决定action在什么情况下运行的处罚条件
	- Prerule、Postrule、Portrule、Hostrule
	- Portrule：检测服务匹配的特征字符串
- Action
	满足条件时的具体操作
	- 条件、循环判断
- `--script-trace`













