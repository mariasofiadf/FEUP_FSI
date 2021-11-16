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

**Objetivo:** Compilar o programa stack.c.

**Procedimento:** Compilar o programa vulnerável, desligando a <em>StackGuard<em> e a proteção de stack não executável. Os comandos necessários para a compilação já estavam incluídos no makefile. Assim, foi apenas necessário escrever make para a execução desses comandos.

**Conclusão:** Observando o programa stack.c, é possível perceber que este abre um ficheiro de input e copia 517 bytes para um char str[517]. É depois chamada a função bof, que recebe um apontador para char. Dentro desta função é usado strcpy que não verifica o tamanho da string a copiar. <br>
Como são alocados apenas 100 bytes para a variável local buffer, e a string de input pode ter até 517 bytes, é possível causar buffer overflow e reescrever o endereço de retorno da função.

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