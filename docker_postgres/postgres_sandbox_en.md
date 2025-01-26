# Deploying Your Docker Postgres Development Solution for Data Practice, Sandboxing, and Beyond  

Whether you're isolating applications, experimenting without consequences, or avoiding software bloat on your local machine, running PostgreSQL in a Docker container offers significant advantages. But it’s easy to stumble into headaches during setup—I learned this the hard way when I first tried deploying it, expecting simplicity akin to installing SQL Server locally or on Azure. The good news? I’ve cracked the code and am sharing a ready-to-use `docker-compose` setup to save you time and frustration.  

---

## **Prerequisites**  
- Docker installed ([Get Docker](https://www.docker.com/get-started))  
- Basic familiarity with Docker concepts (images, containers, volumes, networks)  

---

## **Why Docker + PostgreSQL?**  
A Docker container encapsulates software environments, ensuring consistent behavior across machines. Paired with PostgreSQL—a powerful relational database for web apps, analytics, and data science—this duo streamlines learning and development.  

### Key Benefits:  
- **Simplified Setup**: Skip manual PostgreSQL installation. Pull a preconfigured Docker image and launch with one command.  
- **Portability & Reproducibility**: Share containers across teams or machines without losing configurations or data.  
- **Security & Isolation**: Shield your OS from database vulnerabilities. Run PostgreSQL in isolated networks alongside tools like Jupyter or Dash.  
- **Scalability**: Adjust CPU, memory, and storage on demand. Spin up multiple containers for load testing or version comparisons.  

---

## **Step-by-Step Deployment**  

### 1. Configure Persistence and Backups  
To ensure data survives container restarts or updates, map a local directory to the container’s storage. For this tutorial, we’ll use a local volume (`./db_sql`) and recommend automating backups via cron jobs.  

> **Pro Tip**: In production, consider Docker-managed volumes or cloud storage for better scalability.  

---

### 2. Create the `docker-compose.yml` File  
Copy the configuration below into a file named `docker-compose.yml`:  

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:latest
    container_name: postgresql
    environment:
      POSTGRES_DB: mydb       # Default database name
      POSTGRES_USER: myuser   # Admin username
      POSTGRES_PASSWORD: mypassword  # Admin password
    volumes:
      - ./db_sql:/var/lib/postgresql/data  # Persist data to local directory
    ports:
      - "5432:5432"  # Map host port 5432 → container port 5432
    networks:
      - pg-network  # Isolated network for secure communication

  pgadmin:
    image: dpage/pgadmin4:latest  # Web-based PostgreSQL admin tool
    container_name: pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@pgadmin.com    # Login email
      PGADMIN_DEFAULT_PASSWORD: adminpassword     # Login password
    ports:
      - "4321:80"  # Access PgAdmin at http://localhost:4321
    networks:
      - pg-network

networks:
  pg-network:
    driver: bridge  # Enable communication between containers
```

---

### 3. Launch the Containers  
Open a terminal, navigate to the directory containing `docker-compose.yml`, and run:  
```bash
docker-compose up -d  # Start in detached mode
```  
Wait for Docker to pull the images and initialize the containers.  

![Container startup logs](https://github.com/pedropberger/tutorials/assets/98188778/71cead1b-ebe8-4dc7-8525-e5ca7296b0f3)  

---

### 4. Access PgAdmin  
Visit **http://localhost:4321** (or replace `localhost` with your machine’s IP if accessing remotely).  

---

### 5. Connect PostgreSQL to PgAdmin  
1. **Log in to PgAdmin**:  
   - Email: `admin@pgadmin.com`  
   - Password: `adminpassword`  

2. **Register the PostgreSQL Server**:  
   - Right-click **Servers** → **Create** → **Server...**  
   - Under **General**, name your server (e.g., "Docker PostgreSQL").  
   - Under **Connection**:  
     - **Host**: `postgres` (uses Docker’s internal DNS)  
     - **Port**: `5432`  
     - **Maintenance Database**: `mydb`  
     - **Username/Password**: `myuser` / `mypassword`  

![PgAdmin server registration](https://github.com/pedropberger/tutorials/assets/98188778/03946a28-7f93-4660-9a25-e41fe7b4c90f)  

3. **Explore Your Database**:  
   Expand the server in the sidebar to view tables, run queries, or manage users.  

---

## **Next Steps**  
- **Automate Backups**: Add a cron job to periodically back up `./db_sql`.  
- **Integrate with Tools**: Connect this PostgreSQL instance to Apache Airflow, Python scripts, or BI tools like Metabase.  
- **Scale Securely**: For production, replace hardcoded credentials with Docker secrets or environment variables.  

---

## **Troubleshooting Tips**  
- **Connection Issues?** Ensure both containers are running (`docker ps`) and on the same network.  
- **Data Not Persisting?** Verify the `./db_sql` directory exists and has write permissions.  

---

By containerizing PostgreSQL, you’ve built a flexible, disposable environment perfect for experimentation. Now, go break things—your local machine will thank you!  

**Ready to level up?** Learn how to orchestrate this setup with Airflow [here](#).
