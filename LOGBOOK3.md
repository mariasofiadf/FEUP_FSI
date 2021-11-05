
# Trabalho realizado na Semana #3

https://research.nccgroup.com/2020/07/12/understanding-the-root-cause-of-f5-networks-k52145254-tmui-rce-vulnerability-cve-2020-5902/
CVE-2020-5902 was disclosed on July 1st, 2020 by F5 Networks in K52145254 as a CVSS 10.0 remote code execution vulnerability in the Big-IP administrative interface

https://support.f5.com/csp/article/K52145254
This vulnerability allows for unauthenticated attackers, or authenticated users, with network access to the Configuration utility, through the BIG-IP management port and/or self IPs, to execute arbitrary system commands, create or delete files, disable services, and/or execute arbitrary Java code. This vulnerability may result in complete system compromise. The BIG-IP system in Appliance mode is also vulnerable. This issue is not exposed on the data plane; only the control plane is affected.
K52145254: TMUI RCE vulnerability CVE-2020-5902
The Traffic Management User Interface (TMUI), also referred to as the Configuration utility, has a Remote Code Execution (RCE) vulnerability in undisclosed pages. (CVE-2020-5902)
Product-> BIG-IP (LTM, AAM, Advanced WAF, AFM, Analytics, APM, ASM, DDHD, DNS, FPS, GTM, Link Controller, PEM, SSLO, CGNAT)

https://swarm.ptsecurity.com/rce-in-f5-big-ip/
The CVE-2020-5902 vulnerability has been assigned a CVSS score of 10, the highest possible. According to the Threat Intelligence Services of Positive Technologies, before the fixes there were more than 8,000 devices available on the Internet and vulnerable to this issue.
Conclusion

We were able to get Remote Command Execution on the F5 Big-IP appliance via the next three easy steps:

    Discovering a misconfiguration of the Apache HTTP Server and Apache Tomcat
    Discovering the use of default credentials for HSQLDB
    Discovering questionable static methods in the F5 Big-IP TMUI libraries

The timeline:

    1 April, 2020 — Reported to F5 Networks
    3 April, 2020 — Vulnerability reproduced by F5
    1 July , 2020 — Security Advisory and Fixes have been released


https://badpackets.net/over-3000-f5-big-ip-endpoints-vulnerable-to-cve-2020-5902/

## Identificação

 - CVE-2020-5902 foi divulgado como sendo uma vulnerabilidade de execução de código remoto.
 - Um produto afetado por esta vulnerabilidade foi o BIG-IP <em>administrative interface<em>. A componente afetada foi <em>The Traffic Management User Interface (TMUI) /Configuration utility<em>.
 - item3
 - item4

## Catalogação

- CVE-2020-5902 foi divulgado a 1 de julho, 2020 por F5 Networks
- item2
- item3
- item4

## Exploit

- item1
- item2
- item3
- item4

## Ataques

- item1
- item2
- item3
- item4
