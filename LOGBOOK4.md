# Trabalho realizado na Semana #4

## Seed Labs

## Tarefa 1

 - Tal como esperado foi possível criar, observar e apagar variáveis de ambiente.

## Tarefa 2

 - Passo 1 – As variáveis de ambiente são impressas para o ficheiro pelo processo filho.
 - Passo 2 – As variáveis de ambiente são impressas para o ficheiro pelo processo pai.
 - Passo 3 – Quando um processo filho é criado através de um fork, este herda as variáveis de ambiente do processo pai. Assim, a execução dos dois primeiros Passos, levam ao mesmo output. 

## Tarefa 3
 - Passo 1 - Não há qualquer output ao correr o programa.
 - Passo 2 - As variáveis de ambiente são impressas.
 - Passo 3 - int execve(const char *pathname, char *const argv[],  char *const envp[]);
    envp é um array de apontadores para strings, que são passados como ambiente para o novo programa. Sendo assim, quando envp = NULL, no nosso myprintenv.c, o novo programa não terá nenhuma variável de ambiente 

## Tarefa 4

Tal como esperado, a variáveis de ambiente do programa que chama system() são passadas ao novo programa /bin/sh.

## Tarefa 5

- Passo 1 - Assim que a variável de ambiente PATH é alterada, não é possível definir mais nenhuma, uma vez que, estando o PATH alterado, não é possível aceder ao principal diretório de comandos executáveis do sistema (/usr/bin/). Consequentemente, deixa de ser possível correr o nosso (ou qualquer) programa no mesmo terminal.

 - Passo 2 - Com o comando "ls -l" é possível confirmar que o dono do programa foi corretamente alterado para root e também que o setuid está ativo, pois o setgid bit está com "s":
   - -rwsr-xr-x 1 root seed 16768 Nov 11 05:18 task5


 - Passo 3 - Pelas razoẽs descritas no passo anterior, a variável PATH não é modificada. No entanto, as novas variáveis que foram definidas, são passadas para o processo filho, <em>Set-UID<em>. A variável LD_LIBRARY_PATH não se encontra nas variáveis de ambiente impressas pelo nosso programa nem pela execucão de env.


## Tarefa 6

 - Correndo o programa, sem alterar a variável de ambiente PATH, é corrido o /bin/ls e são impressos no terminal todos os ficheiros/diretórios do diretório atual, tal como esperado.

 - Quando a variável PATH é alterada para conter /home/seed:PATH no seu início, o programa irá correr o programa /home/seed/ls em vez de correr /bin/ls como seria desejado pelo programador. 

 - Como o programa original corre como root e chama o nosso programa ls, este segundo também irá correr como root. Sendo assim é possível obter controlo sobre a máquina e correr o que desejarmos.
 
 - Nota: é possível verificar que, quando chamado pelo programa original (que tem SetUID ativo), /home/seed/ls corre como root, usando a função geteuid().

## CTF

## Reconhecimento

 - Wordpress versão 5.8.1
 - WooCommerce plugin versão 5.7.1
 - Booster for WooCommerce plugin 5.4.3
 - Utilizadores: admin e Orval Sanford

## Pesquisa por vulnerabilidades e Escolha

Ao pesquisar por vulnabilidades para este software e respetivos plugins, nas versões em que se encontram, descobrimos uma vulnerabilidade (CVE-2021-34646) no plugin "Booster for WooCommerce", que permite a um atacante que se autentique como qualquer utilizador (incluindo admin).

## Encontrar um exploit e explorar a vulnerabilidade

Encontramos um exploit em https://www.wordfence.com/blog/2021/08/critical-authentication-bypass-vulnerability-patched-in-booster-for-woocommerce/. <br> O exploit consiste em:
  1. Enviar um request para o <em>home<em> URL com o wcj_user_id a 1, pois esperado que o id de admin seja 1, que é a conta a queremos acesso.
  2. Como o código de verificação para recuperar conta é apenas um hash md5 da hora do pedido, é possível criar o URL que verifica o email.
  3. Através do seguinte script php, geramos vários URL com a hora próxima (diferença de segundos) da hora do pedido, para garantir que um deles tem a hora exata do pedido e, então, o link que verifica o email: <br>
  ![PHP script](/images/logbook4/php.png)
  4. Após aceder ao URL de confirmação de email, passamos a ter acesso como admin.