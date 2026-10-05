# 🔗 Encurtador de URLs

Aplicação web de **encurtamento de URLs** desenvolvida em **Java**, utilizando **Javalin** para a construção da aplicação web, **MongoDB** para persistência dos dados e **Redis** como mecanismo de **cache**, acessado através da biblioteca **Jedis**.

O projeto tem como objetivo demonstrar, de forma simples e prática, como integrar uma aplicação Java com um banco de dados NoSQL orientado a documentos e um sistema de cache em memória.

---

## 🚀 Tecnologias utilizadas

* ☕ **Java**
* 🌐 **Javalin** — framework web para Java
* 🍃 **MongoDB** — persistência dos dados
* ⚡ **Redis** — armazenamento em memória utilizado como cache
* 🔌 **Jedis** — cliente Java para comunicação com o Redis
* 🧩 **HTML/CSS** — interface da aplicação

---

## 🏗️ Arquitetura

A aplicação utiliza dois mecanismos de armazenamento com responsabilidades diferentes:

```text
                 ┌─────────────────┐
                 │     Navegador   │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │     Javalin     │
                 │   Aplicação Web │
                 └────────┬────────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
      ┌─────────────┐           ┌─────────────┐
      │    Redis    │           │   MongoDB   │
      │    Cache    │           │ Persistência│
      └─────────────┘           └─────────────┘
             ▲
             │
          Jedis
```

### Responsabilidades

**MongoDB**

Responsável pelo armazenamento permanente das URLs.

**Redis**

Responsável pelo armazenamento temporário das URLs mais acessadas, evitando consultas repetidas ao MongoDB.

**Jedis**

Biblioteca Java utilizada para estabelecer a comunicação entre a aplicação e o Redis.

**Javalin**

Responsável por receber as requisições HTTP e executar as operações da aplicação.

---

# 🔗 Funcionamento

## 1. Encurtamento

O usuário informa uma URL original:

```text
https://www.exemplo.com.br/artigo/um-artigo-muito-grande
```

A aplicação gera um identificador menor:

```text
abc123
```

E associa as duas informações:

```text
abc123 → https://www.exemplo.com.br/artigo/um-artigo-muito-grande
```

A associação é armazenada no MongoDB.

---

## 2. Acesso à URL encurtada

Quando o usuário acessa:

```text
http://localhost:7070/abc123
```

a aplicação primeiro consulta o Redis.

### Cache HIT

Se a URL estiver no Redis:

```text
Javalin
   │
   ▼
 Redis
   │
   └── URL encontrada
          │
          ▼
      Redirecionamento
```

Nesse caso, o MongoDB não precisa ser consultado.

### Cache MISS

Caso a URL não esteja no Redis:

```text
Javalin
   │
   ▼
 Redis
   │
   └── URL não encontrada
          │
          ▼
       MongoDB
          │
          ▼
       URL encontrada
          │
          ├──────► Redis
          │
          ▼
    Redirecionamento
```

A URL recuperada do MongoDB é colocada no Redis para que os próximos acessos sejam mais rápidos.

---

# ⚡ Estratégia de Cache

O projeto utiliza o conceito de **Cache-Aside**.

O fluxo pode ser representado da seguinte forma:

```text
             ┌──────────────┐
             │   Requisição │
             └───────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │    Redis    │
              └──────┬──────┘
                     │
             ┌───────┴───────┐
             │               │
          HIT│               │MISS
             ▼               ▼
       ┌──────────┐    ┌──────────┐
       │ Retorna  │    │ MongoDB  │
       │   URL    │    └────┬─────┘
       └──────────┘         │
                            ▼
                     ┌─────────────┐
                     │ Redis SET   │
                     └──────┬──────┘
                            │
                            ▼
                       Retorna URL
```

Essa estratégia reduz a quantidade de consultas realizadas no banco de dados.

---

# 🗄️ MongoDB

O MongoDB é utilizado como banco de dados persistente.

Um documento pode possuir uma estrutura semelhante a:

```json
{
    "codigo": "abc123",
    "url": "https://www.exemplo.com.br/artigo",
    "dataCriacao": "2026-10-05"
}
```

A vantagem de utilizar MongoDB nesse projeto é trabalhar diretamente com documentos, sem a necessidade de uma estrutura tradicional baseada em tabelas e relacionamentos.

---

# ⚡ Redis

O Redis armazena temporariamente as associações entre o código da URL e a URL original.

Por exemplo:

```text
CHAVE: abc123
VALOR: https://www.exemplo.com.br/artigo
```

Conceitualmente:

```text
SET abc123 "https://www.exemplo.com.br/artigo"
```

Posteriormente:

```text
GET abc123
```

retorna:

```text
https://www.exemplo.com.br/artigo
```

---

# ☕ Jedis

O **Jedis** é utilizado pela aplicação Java para acessar o Redis.

Um exemplo simplificado:

```java
try (Jedis jedis = new Jedis("localhost", 6379)) {

    jedis.set("abc123", "https://www.exemplo.com.br");

    String url = jedis.get("abc123");

    System.out.println(url);
}
```

Nesse exemplo:

```text
Java
  │
  ▼
Jedis
  │
  ▼
Redis
```

---

# 🌐 Javalin

O Javalin é utilizado para criar as rotas HTTP da aplicação.

Um exemplo conceitual:

```java
app.post("/encurtar", ctx -> {
    // recebe a URL
    // gera o código
    // salva no MongoDB
    // retorna a URL encurtada
});
```

E para acessar uma URL encurtada:

```java
app.get("/{codigo}", ctx -> {
    // consulta Redis
    // se não encontrar, consulta MongoDB
    // redireciona para a URL original
});
```

---

# 📌 Principais operações

## Encurtar uma URL

```text
POST /encurtar
```

Recebe uma URL e gera um código para ela.

Exemplo:

```text
URL:
https://www.ifrs.edu.br/

URL encurtada:
http://localhost:7070/a8F3k
```

---

## Acessar uma URL encurtada

```text
GET /a8F3k
```

A aplicação:

1. Consulta o Redis.
2. Se encontrar a URL, utiliza o valor armazenado no cache.
3. Se não encontrar, consulta o MongoDB.
4. Armazena o resultado no Redis.
5. Redireciona o usuário para a URL original.

---

# 🧠 Conceitos demonstrados

Este projeto pode ser utilizado para estudar diversos conceitos de desenvolvimento de aplicações web:

* Programação Java
* Desenvolvimento Web
* HTTP
* Javalin
* APIs e rotas
* MongoDB
* NoSQL
* Redis
* Cache
* Cache-Aside
* Jedis
* Persistência de dados
* Redirecionamento HTTP
* Integração entre diferentes tecnologias

---

# 🛠️ Pré-requisitos

Para executar o projeto, é necessário possuir:

* Java instalado
* MongoDB em execução
* Redis em execução
* Git

Verifique a instalação do Java:

```bash
java -version
```

Verifique o Redis:

```bash
redis-cli ping
```

O Redis deverá responder:

```text
PONG
```

---

# 📥 Clonando o projeto

```bash
git clone https://github.com/IgorAvilaPereira/encurtadorURL.git
```

Entre no diretório:

```bash
cd encurtadorURL
```

Entre no diretório da aplicação:

```bash
cd encurtador_url
```

---

# ▶️ Executando

Configure o MongoDB e o Redis localmente e execute a aplicação Java.

Após iniciar a aplicação, acesse pelo navegador:

```text
http://localhost:7070
```

> A porta pode ser alterada conforme a configuração da aplicação.

---

# 🔍 Exemplo de funcionamento

Suponha que o usuário informe:

```text
https://www.ifrs.edu.br/
```

A aplicação poderá gerar:

```text
abc123
```

No MongoDB:

```text
abc123 → https://www.ifrs.edu.br/
```

Após o primeiro acesso, a associação também estará disponível no Redis:

```text
Redis

abc123 → https://www.ifrs.edu.br/
```

Nos acessos seguintes:

```text
Navegador
    ↓
Javalin
    ↓
Redis
    ↓
URL encontrada
    ↓
Redirect
    ↓
https://www.ifrs.edu.br/
```

Dessa forma, o MongoDB não precisa ser consultado a cada acesso.

---

# 📊 Cache Hit x Cache Miss

## Cache Hit

A informação está disponível no Redis.

```text
Redis → encontrou → retorna URL
```

É o caminho mais rápido.

## Cache Miss

A informação não está disponível no Redis.

```text
Redis → não encontrou
             ↓
          MongoDB
             ↓
       recupera URL
             ↓
       salva no Redis
             ↓
        retorna URL
```

O próximo acesso poderá ser atendido diretamente pelo Redis.

---

# 🎯 Objetivo didático

O projeto foi desenvolvido com finalidade **didática**, permitindo observar na prática como diferentes tecnologias podem trabalhar conjuntamente em uma aplicação.

A principal ideia é demonstrar que o Redis não substitui necessariamente o banco de dados principal.

Neste projeto:

```text
MongoDB = persistência
Redis   = cache
Jedis   = comunicação Java ↔ Redis
Javalin = aplicação Web
Java    = linguagem de programação
```

Essa separação de responsabilidades permite compreender uma arquitetura bastante comum em aplicações que precisam melhorar o desempenho de consultas frequentes.

---

# 📚 Fluxo completo

```text
                 USUÁRIO
                    │
                    ▼
              ┌───────────┐
              │  Javalin  │
              └─────┬─────┘
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
      ENCURTAR             ACESSAR
          │                   │
          ▼                   ▼
      MongoDB              Redis
                              │
                     ┌────────┴────────┐
                     │                 │
                    HIT               MISS
                     │                 │
                     ▼                 ▼
                  Redirect          MongoDB
                                       │
                                       ▼
                                     Redis
                                       │
                                       ▼
                                    Redirect
```

---

# 👨‍🏫 Autor

**Igor Avila Pereira**

Professor e desenvolvedor.

GitHub:

[https://github.com/IgorAvilaPereira](https://github.com/IgorAvilaPereira)



