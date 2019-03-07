# Desafio Wide Software

Imagine que você é desenvolvedor de um software que gerencia a rede de um grande cliente. Após alguns meses este cliente nos pediu para desenvolver uma funcionalidade nova no sistema, para que equipamentos de fabricantes diferentes possam se comunicar com nossa plataforma de autenticação de rede.

Sabemos que os equipamentos nos enviam os dados através de uma requisição REST, sabemos também que cada equipamento envia para um determinado endpoint, todos os equipamentos nos enviam as seguinte informações:

- MacAddress do equipamento
- MacAddress do dispositivo do usuário
- Username do usuário
- Senha do usuário

Porém temos uma problema, cada equipamento manda a informação em um formato diferente, pois os fabricantes não possuem um padrão. Temos 3 equipamentos na rede do cliente, são eles:

- Fabricante ACME Corp.
- Fabricante Industrias Stark
- Fabricante Umbrella Corp.

Porém temos na documentação de cada equipamento, como eles nos enviam as informações:

## Documentação dos equipamentos

### ACME Corp.

> **[POST]** /acme
```json
{
    "equipament_address": "E4-BF-18-28-93-F6",
    "rocket_address": "4F-E7-DF-20-AE-4C",
    "rocket_user": "user_test",
    "rocket_pwd": "123456"
}
```

### Industrias Stark

> **[POST]** /stark
```json
{
    "network_address": "E4-BF-18-28-93-F6",
    "iron_address": "4F-E7-DF-20-AE-4C",
    "iron_id": "user_test",
    "iron_pwd": "123456"
}
```

### Umbrella Corp.

> [POST] /umbrella
```json
{
    "local_address": "E4-BF-18-28-93-F6",
    "zombie_address": "4F-E7-DF-20-AE-4C",
    "zombie_id": "user_test",
    "zombie_pwd": "123456"
}
```

## Requisitos do sistema

### Requisito 1
Desta forma nosso sistema deverá possuir 3 enpoints:
 - **[POST]** /acme
 - **[POST]** /stark
 - **[POST]** /umbrella

### Requisito 2
Como cada equipamento nos envia os dados em um formato diferente porém são as mesmas informações, devemos abstrair para uma classe de domínio chamada "NetworkData", como no exemplo abaixo:

```java
class NetworkData {
    private String deviceMacaddress;
    private String userMacaddress;
    private String username;
    private String password;

    // ... code
}
```
### Requisito 3
Após criar o objeto "NetworkData" deverá  ser feito uma requisição para nosso microserviço de usuários para validar se o usuário existe e poderá se autenticar.

> [POST] http://dev.widesoftware.com.br/guest

> HEADER Authorization: <access_token>
```json 
{
    "deviceMacaddress": "00-00-00-00-00-00",
    "userMacaddress": "00-00-00-00-00-00",
    "username": "user_test",
    "password": "123456"
}
```
O retorno pode ser:

> HTTP Status **204** em caso de sucesso

> HTTP Status **401** em caso de acesso negado

### Requisito 4
Em caso de sucesso na autenticação devemos enviar os dados do objeto "NetworkData" para uma fila, para que outros serviços possam efetuar uma série de processamentos posteriormente.

> [POST] http://dev.widesoftware.com.br/auth-success-queue

> HEADER Authorization: <access_token>
```json 
{
    "deviceMacaddress": "00-00-00-00-00-00",
    "userMacaddress": "00-00-00-00-00-00",
    "username": "user_test",
    "password": "123456"
}
```

### Requisito 5
Após processar a autenticação e enviar os dados para a fila devemos retornar a seguinte resposta para o equipamento:

#### Em caso de sucesso:
> HTTP Status 200
```json
{
    "message": "ok"
}
```
#### Em caso de falha na autenticação
> HTTP Status 401
```json
{
    "message": "Invalid credentials"
}
```