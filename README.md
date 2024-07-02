# PostgreSQL Advanced

- Esse documento é mais recomendado para quem quer saber um pouco mais sobre esse banco de dados passando por alguns topicos mais avançados, mas mantendo sempre facil para todos entender :D

## Overview
- Criado em 1986
- Crescendo em popularidade

![Logo](./images/ranking.jpg)

- <b>psql</b> é a CLI para interagir com o SGBD
- bom para saber os comandos pois ele tem auto complete

## Basic Architecture and Terminology
- <b>Cluster</b>: é um postgres em um server, onde temos usuários permissões
- <b>Database</b>:  é o armazenamento onde o seu arquivo estará com tabelas e indicies
- <b>User</b>: quem vai interagir e possui permissões
- o PostgreSQL utiliza um arquitetura do tipo <b>Client-Server</b>  onde o cliente estabelece sua conexão com o servidor de banco de dados

![ClientServer](./images/clientserver.png)

- Ele armazena suas conexões no server process para sua sessões para cada conexão, é possível ver isso com `show max_connections`
- Postgres não vem com um pool de connections por default, você pode utilizar o <b>PgBouncer</b>
A memória no PostgreSQL pode ser dividido em duas categorias, <b>Shared Memory</b> e <b>Local Memory</b>
- <b>Local</b>: Usado para um unico usuario
- <b>Shared</b>: Compartilhado com todos server process
- <b>Memory: Shared Buffers</b>: um dos topicos mais importantes do postgres, ele mantem uns dados na memoria para busca rapidas, por default é 128MB, é possivel ver ele pelo comando `show shared_buffers`. Resumindo um "Cache", um exemplo é quando voce executa a query segunda vez e vai muito mais rapido
- <b>Local Memory</b>: é alocado para cada server process, ele tem algumas areas como `work_mem` que é usado para sort e hash table (default 1MB), a segunda area é a `temp_buffers` onde armazena tabelas temporarias (default 8MB) e a terceira area é a `maintenance_work_mem` onde fica o [vacuum](https://www.postgresql.org/docs/current/sql-vacuum.html) e operacões de criação de indicies. 
- <b>WAL: Write Ahead Log</b> é um log de todas as mudanças de tabelas e indicies, ela é usada em caso de desastres com o banco de dados, mas tambem uma forma de voce replicar os dados para suas replicas. Após configurar o servidor das replicas em seus arquivos de configuração ele envia via TCP o WAL para as replicas. Replicações podem ser assincronas e sincronas (com locks de recurso até a confirmação da replicação), e podemos ter replicações logicas.

## Common Administrative Tasks

### Manage Tables

- `IF EXISTS OR NOT EXISTS` é sempre bom para evitar erros
- podemos usar `CREATE UNLOGGED TABLE ...` para criar dados que não vão ser enviados para o WAF
    - as coisas podem ficar mais rapidas por conta da sincronização de replicas, mas como auto-explicativo, o dado não vai ser replicado
- `CREATE VIEW ...` é basicamente uma query com um nome, os resultados não são materializados
    - depois elas são acessadas via query normalmente
- `\d+` para inspecionar as tabelas com o psql

### Schemas

- <b>schemas</b> são basicamente namespaces dentro de um banco de dados, o default é chamado de `public`. Voce pode ter tabelas,views diferentes para cada schema
- <b>database objects</b> são basicamente tabelas, views, sequences e por ai vai...
- <b>search path</b> é uma lista de schemas que o banco de dados vai procurar data objects que não foram referenciados na query por um schema, por exemplo `SELECT * FROM xpto` ele vai usar o search path para saber qual schema esta essa tabela xpto
- schemas podem ser uteis para multi-tenancy users (separar cada ambiente pra cada cliente), mas voce pode ter overhead para manter isso
- o schema `pg_catalog` possui varias tabelas e algumas delas são muito importante para ate conseguir retirar metricas do seu banco de dados, alguns exemplos:
![pgcatalog](./images/pgcatalog.png)

### Data Integrity

- postgres podem ter tipos de dados comuns e dados especiais
- Comuns
![comons](./images/common.png)
- Especiais
![special](./images/special.png)
- saber quais dados voce vai salvar é super importante tanto para saber a quantidade de dados que voce vai salvar, por exemplo 10GB por dia (pode ser ate requisito de System Design Interview)
- quanto para evitar erros basicos de tipos
- utilizar `NOT NULL` constraint para colocar dados obrigatorios
- utitlizar <b>unique constraint</b> para dados que não podem ser duplicados, o Postgres cria por baixo dos panos um b-tree index para verificar se essa unique constraint ja existe em outro lugar
- um comando de exemplo `ALTER TABLE users ADD CONSTRAINT users_name_uk UNIQUE(name);`

### Users and Privileges

- Um bom caso de uso para privileges é criar privileges para cada replicas caso voce queira replicas read-only ou replicas com read-write
- é possivel restringindo aquele usuario de comandos de escrita
- <b>role</b> é um jeito de agrupar os usuarios, e poder ter priveleges para essa role por exemplo
- é possivel dar permissões com o comando `GRANT`


## Query Optimization with Indexes