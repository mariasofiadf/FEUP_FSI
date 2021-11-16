# Trabalho realizado na Semana #5

## **Seed Labs**

## Tarefa 1
**Objetivo:** Correr o shellcode de 32-bit e 64-bit e descrever o ocorrido.

**Conclusão:** Tal como esperado, o programa setUID corre o shellcode e executa /bin/sh, abrindo a shell com privilégios de root.

## Tarefa 2
**Objetivo:** Compilar o programa stack.c.

**Conclusão:** Observando o programa stack.c, é possível perceber que este abre um ficheiro de input e copia 517 bytes para um char str[517]. É depois chamada a função bof, que recebe um apontador para char. Dentro desta função é usado strcpy que não verifica o tamanho da string a copiar. <br>
Como são alocados apenas 100 bytes para a variável local buffer, e a string de input pode ter até 517 bytes, é possível causar buffer overflow e reescrever o endereço de retorno da função