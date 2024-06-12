# 信息收集

## 前言

信息收集是渗透测试的前期工作，目的在于扩大目标的攻击面，为后续有针对性的测试做铺垫。笔者认为，**信息收集=流程+工具**，有了完整的流程加上趁手的工具，信息收集就可以实现~~自动化（~~无脑化），故整理了一套方法论，供读者参考。

以下内容摘自笔记，语言为英语，是因为实际工作中，我们必须时刻注意保护自己，尽可能减少特征（当然下面的笔记中有很明显的中国特征，是个错误示范）。比如工作设备不能被恶意的物理接触（e.g. u盘）、操作系统的时区、语言、敏感文件的存储、特定虚拟机只做特定的事、网络环境等等，这些内容由于与本 blog 不相关故不展开。

## 流程

> 前提：假定已知某个目标域名

* whois
* fuzzing top domain name
  * host="xxx" in fofa
  * search which domain has been taken in godaddy
* use tool to search specific target
  * fofa clue search
  * whois(whoxy, sqlsec.com)
    * similar domains
    * whois history
  * github.com: in:xxx
  * virustotal
  * CMS fingerprint
  * censys
  * netlas.io
  * beian.miit.gov.cn
  * error log
* find real ip
  * favicon hash
  * DNS record
  * ssl certificate
  * pingning from many places
  * search specific body
  * `ctrl + u` → related code → directory travel(try to change ip when acces denied)
  * nslookup
* scan related network address(e.g., 1.1.1.1 - 1.1.1.10)
  * don't miss website can't be access(state code: 3xx \~ 4xx)
* find all subdomain
* scan port && directory(special / rarely seen port or directory may match with specific exploitation)
  * duplicate
  * attack according to type
  * wanna dynamic file & admin interface, nor static file(check source code)
    * audit code and search "admin, login, ajax"
    * chrome developer tool
* discover bug & exp
  * login interface without verification, try **brute force**

## 工具

Register:

* [https://strongpassword.onionmail.org](https://strongpassword.onionmail.org/)
* [https://10minutemail.com](https://10minutemail.com/)

ip check:

* virustotal
* ip address: [https://pry.sh](https://pry.sh/)
* ASN: [https://ipinfo.io/AS3462](https://ipinfo.io/AS3462)
* ping from many places: [https://tools.ipip.net/newping.php](https://tools.ipip.net/newping.php)
* except CDN: [https://www.cnblogs.com/qiudabai/p/9763739.html](https://www.cnblogs.com/qiudabai/p/9763739.html)
* favicon hash, body
* dns record
  * [https://sitereport.netcraft.com/](https://sitereport.netcraft.com/)
  * [https://dnslytics.com](https://dnslytics.com/)
  * [https://passivedns.mnemonic.no](https://passivedns.mnemonic.no/)
* ssl certificate: search.censys.io
* engine: O.zone

passwd guess:

* online: Medusa, Hyrda
  * `medusa -h <ip_address> - admin -p admin -M 10443`
*   offline: Hashcat

    > You cannot use hashcat to recover online accounts (like Gmail, Instagram, Facebook, Twitter, etc.), because hashcat has no way to work on online accounts.

port scan(/24 all are results):

*   masscan, blackwater(**comprehensive** use, adjust **parameter**)

    ```
    blackwater -i x.x.x.x -c 400 -р 1-65535
    ​
    masscan -p1-65535 x.x.x.x/24 --exclude x.x.x.x-x.x.x.x --rate 800 > <result>
    ```
* Nmap(recognize fingerprint): `nmap -A <ip_address> -o ‹output_filename>`
*   rustscan:

    ```
    alias rustscan='docker run -it --rm --name rustscan rustscan/rustscan: 2.1. 1'
    rustscan -a 127.0.0.1 -q --range 1-10000
    ```

bug scan: Nessus, Core Impact, Netsparker

Register:

* [https://strongpassword.onionmail.org/](https://strongpassword.onionmail.org/)
* [https://10minutemail.com/](https://10minutemail.com/)

structure: Metasploit

directory(self-built websites && special path):

*   dirmap:

    ```
    python3 dirmap.py -iF targets.txt -1cf
    python3 dirmap.py -i <url> -lcf
    ```
*   dirsearch:

    ```
    python dirsearch.py -e php -u <url> --exclude-status 403,401 --deep-recursive -o <output_filename>
    ```
* dirb: `dirb <url> -0 <result>`

subdomain:

* oneforall(directory, parameter): `python3 oneforall.py --target <domain> --exclude www run --output`` ``<output_filename>`
* knockpy: `knockpy -d <domain> --recon --bruteforce --save report`
* google: `site: xxxxxx -www`
* fofa: domain="xxxxxx"
* [https://scan.javasec.cn](https://scan.javasec.cn/)
* [https://api.subdomain.center/?domian=](https://api.subdomain.center/?domian=)

search: fofa syntax

traffic listening: burp suite

http probe: [https://github.com/projectdiscovery/httpx](https://github.com/projectdiscovery/httpx)

regular express: [https://ihateregex.io/expr/url/](https://ihateregex.io/expr/url/)

picture: exiftool

version recognize: hash value && \* match

sql injection:

* manually
  * Determine whether there is injection and whether the injection is character or numeric(difference: `quotation marks` && `number of sql`)
    * trick: `1' or 1 = 1 #`, `1 or 1 = 1 #` → retum messge?
    * `sleep`
    * Guess the number of fields in the SQL query statement
    * Determine the order of displayed fields
  * Get target
    * current database
    * table
    * field names in the table
* `sqlmap`
* Defense
  * POD: user input is bound to the statement, separating it from the SQL query
  * Make sure only 1 result is returned
  * Anti-CSRF token
