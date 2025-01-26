**Tutorial: Gerenciando Bancos de Dados PostgreSQL no Docker**

Neste tutorial, exploraremos como configurar e gerenciar bancos de dados PostgreSQL em um container Docker. Você aprenderá a executar comandos básicos, criar bancos de dados, tabelas, usuários e gerenciar permissões de forma prática.

---

### **1. Configurando o PostgreSQL no Docker**

Antes de começar, é essencial ter o Docker instalado. Com ele, você pode executar o PostgreSQL isoladamente em um container. Use o comando abaixo para baixar a imagem oficial:

```bash
docker pull postgres
```

**Criando o Container**  
Execute o seguinte comando para iniciar um container chamado `postgresql`:

```bash
docker run --name postgresql -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres
```

- `--name postgresql`: Define o nome do container.
- `-e POSTGRES_PASSWORD=postgres`: Configura a senha do usuário padrão `postgres`.
- `-p 5432:5432`: Mapeia a porta do container para a mesma porta na máquina local.
- `-d`: Executa o container em segundo plano.

Verifique se o container está ativo com:

```bash
docker ps
```

---

### **2. Acessando o PostgreSQL via Docker Exec**

Para interagir com o PostgreSQL dentro do container, use `docker exec`. O comando abaixo inicia o cliente `psql` como o usuário padrão `postgres`:

```bash
docker exec -it postgresql psql
```

O prompt `postgres=#` indica que você está conectado ao banco de dados padrão.

---

### **3. Criando Bancos de Dados e Tabelas**

**Criando um Novo Banco de Dados**  
No prompt do `psql`, crie um banco de dados chamado `gera_certidao`:

```sql
CREATE DATABASE gera_certidao;
```

**Conectando a um Banco de Dados**  
Para acessar o banco recém-criado, use:

```bash
docker exec -it postgresql psql -U myuser -h localhost -p 5432 gera_certidao
```

**Criando uma Tabela**  
No banco `gera_certidao`, crie a tabela `certidao`:

```sql
CREATE TABLE certidao (
  id SERIAL PRIMARY KEY,
  numero VARCHAR(255) NOT NULL,
  data_emissao TIMESTAMP NOT NULL,
  tipo VARCHAR(255) NOT NULL,
  conteudo TEXT NOT NULL
);
```

Verifique a estrutura da tabela com:

```sql
\d certidao
```

---

### **4. Gerenciando Usuários e Permissões**

**Criando um Novo Usuário**  
Crie um usuário específico para o banco `gera_certidao`:

```sql
CREATE USER user_geracertidao WITH PASSWORD 'g3rac3rtida0';
```

**Concedendo Privilégios**  
- **Privilégios no Banco de Dados**:  
  Para permitir que o usuário gerencie o banco `mpes` (observação: ajuste o nome conforme necessário):

  ```sql
  GRANT ALL PRIVILEGES ON DATABASE mpes TO user_geracertidao;
  ```

- **Privilégios em Tabelas**:  
  Conceda acesso às tabelas do `gera_certidao`:

  ```sql
  -- Permissões no banco
  GRANT SELECT, UPDATE ON DATABASE gera_certidao TO user_geracertidao;

  -- Permissões em todas as tabelas
  GRANT SELECT ON ALL TABLES IN DATABASE gera_certidao TO user_geracertidao;

  -- Permissões no esquema público
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO user_geracertidao;
  ```

**Verificando Permissões**  
Liste os usuários e seus papéis com:

```sql
\du
```

---

### **5. Comandos Úteis do psql**

- `\l`: Lista todos os bancos de dados.
- `\c [nome_banco]`: Conecta a um banco específico.
- `\dt`: Lista tabelas do banco atual.
- `\du`: Lista usuários e papéis.

---

### **6. Conclusão**

Neste tutorial, você aprendeu a executar o PostgreSQL em um container Docker, criar bancos de dados, tabelas e gerenciar permissões de usuários. Esse fluxo é ideal para ambientes de desenvolvimento, garantindo isolamento e facilidade de configuração. Para ambientes de produção, considere ajustes como volumes persistentes e políticas de segurança mais rigorosas.

**Próximos Passos**:
- Configure volumes para persistência de dados.
- Explore backups com `pg_dump`.
- Aprofunde-se em roles e políticas de acesso do PostgreSQL.

Com esses conhecimentos, você está preparado para gerenciar bancos de dados PostgreSQL de forma eficiente usando Docker! 🐳🔍
