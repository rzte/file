# retire.js

retire.js可帮助检测具有已知漏洞的js-library版本（web应用、node应用）

支持：

- [命令行](https://github.com/RetireJS/retire.js/tree/master/node)
- 浏览器插件（[chrome](https://github.com/RetireJS/retire.js/tree/master/chrome)、[firefox](https://github.com/RetireJS/retire.js/tree/master/firefox)）
- [Burp / OWASP Zap插件](https://github.com/h3xstream/burp-retire-js)
- [grunt plugin](https://github.com/bekk/grunt-retire)、[gulp task](https://github.com/RetireJS/retire.js#user-content-gulp-task)

```bash
$ git clone https://github.com/RetireJS/retire.js.git
$ cd retire.js.git
$ npm install -g retire
$ retire
```

安装完成后，在程序的根目录执行`retire`即可

```bash
[rz@/]$ cd /opt/blog/
[rz@/opt/blog]$ retire
retire.js v2.0.2
Loading from cache: https://raw.githubusercontent.com/RetireJS/retire.js/master/repository/jsrepository.json
Loading from cache: https://raw.githubusercontent.com/RetireJS/retire.js/master/repository/npmrepository.json
/opt/blog/app/static/js/jquery.min.js
 ↳ jquery 1.11.1
jquery 1.11.1 has known vulnerabilities: severity: medium; issue: 2432, summary: 3rd party CORS request may execute, CVE: CVE-2015-9251; https://github.com/jquery/jquery/issues/2432 http://blog.jquery.com/2016/01/08/jquery-2-2-and-1-12-released/ https://nvd.nist.gov/vuln/detail/CVE-2015-9251 http://research.insecurelabs.org/jquery/test/ severity: medium; CVE: CVE-2015-9251, issue: 11974, summary: parseHTML() executes scripts in event handlers; https://bugs.jquery.com/ticket/11974 https://nvd.nist.gov/vuln/detail/CVE-2015-9251 http://research.insecurelabs.org/jquery/test/
/opt/blog/app/static/markdown/js/jquery.min.js
 ↳ jquery 1.11.1
jquery 1.11.1 has known vulnerabilities: severity: medium; issue: 2432, summary: 3rd party CORS request may execute, CVE: CVE-2015-9251; https://github.com/jquery/jquery/issues/2432 http://blog.jquery.com/2016/01/08/jquery-2-2-and-1-12-released/ https://nvd.nist.gov/vuln/detail/CVE-2015-9251 http://research.insecurelabs.org/jquery/test/ severity: medium; CVE: CVE-2015-9251, issue: 11974, summary: parseHTML() executes scripts in event handlers; https://bugs.jquery.com/ticket/11974 https://nvd.nist.gov/vuln/detail/CVE-2015-9251 http://research.insecurelabs.org/jquery/test/
```

# DependencyCheck

DependencyCheck可用来检查项目中是否使用了存在已知漏洞的依赖项。目前支持**Java**、**.NET**，并为Ruby、NodeJs、Python以及C/C++(autoconf、cmake)提供了额外的实验性支持

- [Ant Task](https://jeremylong.github.io/DependencyCheck/dependency-check-ant/index.html)
- [Command Line Tool](https://jeremylong.github.io/DependencyCheck/dependency-check-cli/index.html)
- [Gradle Plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/index.html)
- [Jenkins Plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-jenkins/index.html)
- [Maven Plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/index.html) - Maven 3.1 or newer required
- [SBT Plugin](https://github.com/albuch/sbt-dependency-check)

```bash
$ wget http://dl.bintray.com/jeremy-long/owasp/dependency-check-4.0.2-release.zip

$ dependency-check.sh --project blog -s /tmp/BLOGVULN/*.war -o /tmp/BLOGVULN/VULN/
```

# 参考

> [Top 10-2017 A9-Using Components with Known Vulnerabilities](https://www.owasp.org/index.php/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities)
>
> [retire.js](https://github.com/RetireJS/retire.js)
>
> [DependencyCheck github](https://github.com/jeremylong/DependencyCheck)
>
> [DependencyCheck gitpage](https://jeremylong.github.io/DependencyCheck/index.html#)

