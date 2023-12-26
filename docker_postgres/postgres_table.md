docker exec -it postgresql psql

docker exec -it postgresql psql -U myuser -h localhost -p 5432 mpes

docker exec -it postgresql psql -U myuser -h localhost -p 5432 gera_certidao




CREATE DATABASE gera_certidao;

psql -U myuser -h localhost -p 5432 gera_certidao

CREATE USER user_geracertidao WITH PASSWORD 'g3rac3rtida0';

GRANT ALL PRIVILEGES ON DATABASE mpes TO user_geracertidao;

psql -U myuser -h localhost -p 5432 gera_certidao

CREATE TABLE certidao (
  id SERIAL PRIMARY KEY,
  numero VARCHAR(255) NOT NULL,
  data_emissao TIMESTAMP NOT NULL,
  tipo VARCHAR(255) NOT NULL,
  conteudo TEXT NOT NULL
);


\d certidao

GRANT SELECT, UPDATE ON DATABASE "gera_certidao" TO user_geracertidao;

GRANT SELECT ON ALL TABLES IN DATABASE "gera_certidao" TO user_geracertidao;

GRANT SELECT ON ALL TABLES IN SCHEMA public TO user_geracertidao;


\du



Neste tutorial, vamos aprender como usar o PostgreSQL rodando no docker, um sistema de gerenciamento de banco de dados relacional muito popular e poderoso. Vamos ver como executar comandos básicos para criar e manipular bancos de dados, tabelas e usuários usando o docker exec e o psql, um cliente interativo do PostgreSQL.

Para começar, precisamos ter o docker instalado e rodando na nossa máquina. Também precisamos ter uma imagem do PostgreSQL disponível no docker. Podemos usar o comando docker pull postgres para baixar a imagem oficial do PostgreSQL. Em seguida, podemos criar um container chamado postgresql usando o comando docker run:

docker run --name postgresql -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres

Esse comando cria um container chamado postgresql, define a senha do usuário padrão postgres como postgres, mapeia a porta 5432 do container para a porta 5432 da máquina e roda o container em modo daemon (-d). Podemos verificar se o container está rodando usando o comando docker ps:

docker ps

CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                    NAMES
a1b2c3d4e5f6   postgres   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:5432->5432/tcp   postgresql

Agora que temos o container rodando, podemos executar comandos do PostgreSQL dentro dele usando o comando docker exec. Esse comando permite executar um comando em um container em execução. Para executar comandos do PostgreSQL, precisamos usar o psql, que é um cliente interativo do PostgreSQL que permite executar consultas SQL e outros comandos.

Para entrar no psql como o usuário padrão postgres, podemos usar o seguinte comando:

docker exec -it postgresql psql

Esse comando executa o psql em modo interativo (-it) no container postgresql. O prompt do psql deve aparecer:

psql (13.5 (Debian 13.5-1.pgdg110+1))
Type "help" for help.

postgres=#

Podemos ver que estamos conectados ao banco de dados padrão postgres como o usuário postgres. Podemos usar o comando \l para listar todos os bancos de dados existentes:

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 mpes      | myuser   | UTF8     | en_US.utf8 | en_US.utf8 | 
 gera_certidao | myuser   | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(5 rows)

Podemos ver que existem dois bancos de dados criados pelo usuário myuser: mpes e gera_certidao. Vamos ver como conectar a esses bancos de dados usando o psql.

Para conectar a um banco de dados específico, podemos usar o seguinte comando:

docker exec -it postgresql psql -U myuser -h localhost -p 5432 mpes

Esse comando executa o psql no container postgresql, especificando o usuário (-U myuser), o host (-h localhost), a porta (-p 5432) e o nome do banco de dados (mpes). O prompt do psql deve mudar para indicar que estamos conectados ao banco de dados mpes:

psql (13.5 (Debian 13.5-1.pgdg110+1))
Type "help" for help.

mpes=#

Podemos usar o mesmo comando para conectar ao banco de dados gera_certidao, mudando apenas o nome do banco de dados no final:

docker exec -it postgresql psql -U myuser -h localhost -p 5432 gera_certidao

psql (13.5 (Debian 13.5-1.pgdg110+1))
Type "help" for help.

gera_certidao=#

Agora que sabemos como conectar aos bancos de dados, vamos ver como criar e manipular bancos de dados, tabelas e usuários usando o psql.

Para criar um novo banco de dados, podemos usar o comando CREATE DATABASE seguido do nome do banco de dados. Por exemplo, para criar um banco de dados chamado gera_certidao, podemos usar o seguinte comando:

CREATE DATABASE gera_certidao;

Esse comando deve retornar uma mensagem de sucesso:

CREATE DATABASE

Podemos verificar se o banco de dados foi criado usando o comando \l novamente:

postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 mpes      | myuser   | UTF8     | en_US.utf8 | en_US.utf8 | 
 gera_certidao | postgres  | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(6 rows)

Podemos ver que o banco de dados gera_certidao foi criado e pertence ao usuário postgres. Podemos conectar a esse banco de dados usando o comando psql como vimos antes:

psql -U myuser -h localhost -p 5432 gera_certidao

psql (13.5 (Debian 13.5-1.pgdg110+1))
Type "help" for help.

gera_certidao=#

Para criar uma nova tabela dentro de um banco de dados, podemos usar o comando CREATE TABLE seguido do nome da tabela e das colunas e seus tipos. Por exemplo, para criar uma tabela chamada certidao com as colunas id, numero, data_emissao, tipo e conteudo, podemos usar o seguinte comando:

CREATE TABLE certidao ( id SERIAL PRIMARY KEY, numero VARCHAR(255) NOT NULL, data_emissao TIMESTAMP NOT NULL, tipo VARCHAR(255) NOT NULL, conteudo TEXT NOT NULL );

Esse comando deve retornar uma mensagem de sucesso:

CREATE TABLE

Podemos verificar se a tabela foi criada usando o comando \d seguido do nome da tabela:

\d certidao
                               Table "public.certidao"
    Column    |            Type             | Collation | Nullable |      Default      
--------------+-----------------------------+-----------+----------+-------------------
 id           | integer                     |           | not null | nextval('certidao_id_seq'::regclass)
 numero       | character varying(255)      |           | not null |
 data_emissao | timestamp without time zone |           | not null |
 tipo         | character varying(255)      |           | not null |
 conteudo     | text                        |           | not null |
Indexes:
    "certidao_pkey" PRIMARY KEY, btree (id)

Podemos ver que a tabela certidao foi criada no esquema public e tem as colunas e os tipos que especificamos. Também podemos ver que a coluna id é uma chave primária que é gerada automaticamente usando uma sequência.

Para criar um novo usuário no PostgreSQL, podemos usar o comando CREATE USER seguido do nome do usuário e da senha. Por exemplo, para criar um usuário chamado user_geracertidao com a senha g3rac3rtida0, podemos usar o seguinte comando:

CREATE USER user_geracertidao WITH PASSWORD 'g3rac3rtida0';

Esse comando deve retornar uma mensagem de sucesso:

CREATE ROLE

Podemos verificar se o usuário foi criado usando o comando \du:

\du
                                   List of roles
    Role name    │                         Attributes                         │ Member of 
─────────────────┼────────────────────────────────────────────────────────────┼───────────
 myuser         │ Superuser                                                  │ {}
 postgres       │ Superuser, Create role, Create DB, Replication, Bypass RLS │ {}
 user_geracertidao │                                                            │ {}

Podemos ver que o usuário user_geracertidao foi criado e não tem nenhum atributo especial.

Para conceder privilégios a um usuário sobre um banco de dados ou uma tabela, podemos usar o comando GRANT seguido do tipo de privilégio, da palavra ON, do objeto e do usuário. Por exemplo, para conceder todos os privilégios ao usuário user_gerac
