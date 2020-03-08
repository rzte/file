---
### 信息收集
- **<a href="http://www.freebuf.com/sectool/141544.html">recon-ng</a>**
    使用方式类似Metasploit。recon-ng有侦查（recon）、识别（discovery）、汇报（reporting）和攻击（exploitation）四大模块。
模块加载后可以 show info查看模块的用法。
例：
    **查询子域名**：
        use recon/domains-hosts/bing_domain_web         # 这是用的bing，还可以用google
    **查询某个用户名在那些网站（知名）有注册**：
        use recon/profiles-profiles/profiler
        查看表的命令：show profiles
    **查找网站的敏感文件**
        Use discovery/info_disclosure/interesting_files
    **攻击板块**
        Use exploitation/injection/command_injector
    **报告板块**
        use reporting/html


---
### 漏洞扫描
- **NeXpose**
    Rapid7推出的NeXpose Vulnerability Scanner Community Edition是一款**免费**的漏洞扫描程序。它可与**Metasploit**框架整合。
    NeXpose共享版具有以下特性：
    - 能扫描最多32个IP
    - 漏洞数据库可定期升级
    - 可指定风险评估的优先级
    - 可为改进安全性提供指导建议
    - 可与Metasploit整合
    - 通过网站(http://community.rapid7.com)提供共享版的有关支持
    - 易于部署
    - 可作为免费的初级安全解决方案

    商业版具有更多功能，例如IP数量没有限制，可进行分布式扫描，扫描报告更加灵活，可进行web和数据库应用程序扫描，并有专门的技术支持服务。

    NeXpose由两个部分组成：
    - NeXpose扫描引擎：目标识别和检测漏洞的后台程序。共享版程序只有一个引擎，即本地引擎。
    - NeXpose安全控制台：安全控制台负责与扫描引擎互动，以启动扫描任务并接收扫描结果。控制台还配有可配置、操作扫描引擎的Web接口。

---
- ### Nikto
nikto是一款开源的（GPL）网页服务器**扫描器**，它可用对网页服务器进行全面的多种扫描。

可用来扫描敏感目录：
```
nikto -h 192.168.1.15
```