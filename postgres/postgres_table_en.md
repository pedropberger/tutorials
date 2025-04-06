**Tutorial: Managing PostgreSQL Databases in Docker**

In this tutorial, we’ll explore how to set up and manage PostgreSQL databases in a Docker container. You’ll learn how to execute basic commands, create databases, tables, and users, and manage permissions in a practical way.

---

### **1. Setting Up PostgreSQL in Docker**

Before starting, ensure Docker is installed. With Docker, you can run PostgreSQL in an isolated container. Use the following command to download the official PostgreSQL image:

```bash
docker pull postgres
```

**Creating the Container**  
Run the following command to start a container named `postgresql`:

```bash
docker run --name postgresql -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres
```

- `--name postgresql`: Sets the container name.
- `-e POSTGRES_PASSWORD=postgres`: Configures the password for the default `postgres` user.
- `-p 5432:5432`: Maps the container port to the same port on your local machine.
- `-d`: Runs the container in the background.

Verify that the container is running with:

```bash
docker ps
```

---

### **2. Accessing PostgreSQL via Docker Exec**

To interact with PostgreSQL inside the container, use `docker exec`. The following command starts the `psql` client as the default `postgres` user:

```bash
docker exec -it postgresql psql
```

The `postgres=#` prompt indicates you’re connected to the default database.

---

### **3. Creating Databases and Tables**

**Creating a New Database**  
At the `psql` prompt, create a database named `gera_certidao`:

```sql
CREATE DATABASE gera_certidao;
```

**Connecting to a Database**  
To access the newly created database, use:

```bash
docker exec -it postgresql psql -U myuser -h localhost -p 5432 gera_certidao
```

**Creating a Table**  
In the `gera_certidao` database, create the `certidao` table:

```sql
CREATE TABLE certidao (
  id SERIAL PRIMARY KEY,
  numero VARCHAR(255) NOT NULL,
  data_emissao TIMESTAMP NOT NULL,
  tipo VARCHAR(255) NOT NULL,
  conteudo TEXT NOT NULL
);
```

Check the table structure with:

```sql
\d certidao
```

---

### **4. Managing Users and Permissions**

**Creating a New User**  
Create a dedicated user for the `gera_certidao` database:

```sql
CREATE USER user_geracertidao WITH PASSWORD 'g3rac3rtida0';
```

**Granting Privileges**  
- **Database Privileges**:  
  To allow the user to manage the `mpes` database (note: adjust the database name as needed):

  ```sql
  GRANT ALL PRIVILEGES ON DATABASE mpes TO user_geracertidao;
  ```

- **Table Privileges**:  
  Grant access to tables in `gera_certidao`:

  ```sql
  -- Database permissions
  GRANT SELECT, UPDATE ON DATABASE gera_certidao TO user_geracertidao;

  -- Permissions for all tables
  GRANT SELECT ON ALL TABLES IN DATABASE gera_certidao TO user_geracertidao;

  -- Permissions in the public schema
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO user_geracertidao;
  ```

**Checking Permissions**  
List users and their roles with:

```sql
\du
```

---

### **5. Useful psql Commands**

- `\l`: Lists all databases.
- `\c [database_name]`: Connects to a specific database.
- `\dt`: Lists tables in the current database.
- `\du`: Lists users and roles.

---

### **6. Conclusion**

In this tutorial, you learned how to run PostgreSQL in a Docker container, create databases and tables, and manage user permissions. This workflow is ideal for development environments, ensuring isolation and ease of setup. For production environments, consider additional steps like persistent volumes and stricter security policies.

**Next Steps**:
- Set up volumes for data persistence.
- Explore backups with `pg_dump`.
- Dive deeper into PostgreSQL roles and access policies.

With these skills, you’re ready to efficiently manage PostgreSQL databases using Docker! 🐳🔍
