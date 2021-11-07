# Trabalho realizado na Semana #3


## Identificação

 - CVE-2020-5902 foi divulgado como sendo uma vulnerabilidade de execução não autenticada de código remoto.
 - Um produto afetado por esta vulnerabilidade foi o BIG-IP <em>administrative interface<em>. A componente afetada foi <em>The Traffic Management User Interface (TMUI) /Configuration utility<em>.
 - Esta vulnerabilidade está incluida nas categorias <em>Inclusion of Functionality from Untrusted Control Sphere<em> e <em>Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')<em>.

## Catalogação

- CVE-2020-5902 foi divulgado a 1 de julho, 2020 por F5 Networks.
- Esta vulnerabilidade tem um nível de gravidade (score CVSS) de 10, o máximo possível.

## Exploit

- É conhecido um exploit de execução remota de código, explorado pela <em>TEAMARES<em>.
- Esta vulnerabilidade permite obter dados de um servidor.
- O acesso remoto permite a introdução de código nos servidores.

## Ataques

- De acordo com a equipa de Bad Packets, que explorou esta vulnerabilidade, 3012 IPv4 hosts unicos, em 66 países, eram vulneráveis a CVE-2020-5902 
- Foram descobertos em 635 sistemas autónomos esta vulnerabilidade. Estes eram instituições governamentais, bancos, faculdades, hospitais e grandes empresas, entre outros.
- Ataques que explorem esta vulnerabilidade podem também permitir que atacantes ganhem permissões nas redes afetadas, podendo assim distribuir <em>ransomware<em>.
- item4

## Fontes

- https://research.nccgroup.com/2020/07/12/understanding-the-root-cause-of-f5-networks-k52145254-tmui-rce-vulnerability-cve-2020-5902/
- https://support.f5.com/csp/article/K52145254
- https://swarm.ptsecurity.com/rce-in-f5-big-ip/
- https://badpackets.net/over-3000-f5-big-ip-endpoints-vulnerable-to-cve-2020-5902/
- https://www.criticalstart.com/f5-big-ip-remote-code-execution-exploit-cve-2020-5902/


