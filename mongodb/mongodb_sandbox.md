# Installing and Using MongoDB with Docker

MongoDB is a popular NoSQL database known for its flexibility, scalability, and ease of use. Docker is a containerization platform that simplifies the creation, deployment, and execution of applications in containers. Using MongoDB with Docker is a great way to get started with MongoDB quickly or deploy MongoDB applications in a production environment.

This article will show you how to install and use a Docker container with MongoDB. We will cover the following steps:

- Building and running the Docker container
- Connecting to and inserting data into MongoDB
- Verifying the data using MongoDB Compass

## Building and Running the Docker Container

Before you begin, you need to have Docker installed and running on your system.

The first step is to build the Docker image. A Docker image is a lightweight, standalone, and executable package that includes everything needed to run software, including code, runtime, libraries, environment variables, and configuration files. To build the image, you need a `Dockerfile`. This file contains a set of instructions that Docker will use to build the image.

Here is an example of a `Dockerfile` that can be used to build a MongoDB image:

```Dockerfile
# Use the official MongoDB image as a base
FROM mongo

# Expose MongoDB's default port
EXPOSE 27017
```

This `Dockerfile` instructs Docker to use the official MongoDB image as the base image. It also exposes MongoDB's default port, which is 27017.

To build the image, navigate to the directory where you saved your `Dockerfile` and run the following command:

```bash
docker build -t my-mongodb .
```

This will build the image and tag it as `my-mongodb`. The dot (`.`) tells Docker that the `Dockerfile` is located in the current directory.

Once the image is built, you can run the container. To do this, execute the following command:

```bash
docker run -d --name my-mongodb-container -p 27017:27017 my-mongodb
```

This command runs a new container from the `my-mongodb` image.

- The `-d` flag tells Docker to run the container in detached mode, meaning the container will run in the background.
- The `--name` flag gives the container a name, in this case, `my-mongodb-container`. This is useful for referencing it later.
- The `-p` flag publishes the container's port to the host. In this case, it maps port 27017 from the container to port 27017 on the host. This allows you to connect to the MongoDB instance running in the container from your host machine.

Once the container is running, you can verify that MongoDB is accessible on your local host at port 27017.

## Connecting to and Inserting Data into MongoDB

Once the MongoDB container is running, you can connect to it and insert data. You can use **Mongo Shell** or a tool like **MongoDB Compass** to connect to MongoDB.

In this article, we will use the CLI client that comes with the MongoDB Docker image, which is `mongosh` (Mongo Shell). To access the container, run the following command to enter the container:

```bash
docker exec -it my-mongodb-container mongosh
```

This command executes the `mongosh` command inside the `my-mongodb-container`. The `-it` flag indicates that you want to run the command interactively and allocate a pseudo-TTY, allowing you to interact with the shell.

Once inside the Mongo Shell (`mongosh`), you can execute commands to interact with MongoDB. For example, to create a database, a collection, and add documents, you can use the following commands:

```javascript
// Select (or create) the database
use my_database

// Insert a document into a new collection
db.my_collection.insertOne({
  name: "John",
  age: 30,
  city: "New York"
})

// Insert multiple documents
db.my_collection.insertMany([
  { name: "Mary", age: 25, city: "Los Angeles" },
  { name: "Peter", age: 35, city: "Chicago" }
])

// Confirm the inserted documents
db.my_collection.find()
```

These commands will:

1. Create a new database called `my_database` (if it does not exist) and switch to it.
2. Create a new collection called `my_collection` in the `my_database` database.
3. Insert a single document into the `my_collection` collection with the fields `name`, `age`, and `city`.
4. Insert multiple documents into the `my_collection` collection, each with similar fields.
5. Display all documents in the `my_collection` collection to confirm the insertions.

To exit the Mongo Shell, type `exit` and press Enter.

## Verifying the Data

If you want to verify the data using a graphical client like **MongoDB Compass**, you can follow these steps:

1. **Install MongoDB Compass:** Download and install MongoDB Compass from the official MongoDB website.
2. **Connect to MongoDB:** Open MongoDB Compass and create a new connection. In the connection URI field, enter `mongodb://localhost:27017`. Click "Connect".
3. **Navigate to the database and collection:** Once connected, you will see a list of databases. Click on the `my_database` database. Inside it, you will find the `my_collection` collection. Click on it to view the documents you inserted earlier.

Now you should be able to see the data you inserted into MongoDB.

## Conclusion

This article showed how to install and use a Docker container with MongoDB. We also demonstrated how to connect, insert, and verify data in MongoDB using the Mongo Shell and MongoDB Compass.

Using MongoDB with Docker is a great way to get started with MongoDB quickly or deploy MongoDB applications in a production environment. By using Docker, you can easily create, deploy, and run MongoDB instances without worrying about the installation and configuration process. This can save a lot of time and effort, especially if you are working with multiple MongoDB environments.

In addition to the benefits mentioned above, using MongoDB with Docker also provides the following advantages:

- **Portability:** Docker containers are portable, meaning they can run on any machine that has Docker installed. This makes it easy to move MongoDB applications between different environments, such as development, testing, and production.
- **Scalability:** Docker containers can be easily scaled up or down, allowing you to add or remove MongoDB instances as needed to meet your application's requirements. This can be especially useful for applications that experience varying traffic loads.
- **Isolation:** Docker containers provide isolation between applications, meaning one application cannot interfere with another. This can be useful for security and stability.

Overall, using MongoDB with Docker is an excellent way to simplify the development and deployment process of MongoDB applications. If you are new to MongoDB or Docker, I encourage you to give it a try. You may find that it is a great way to improve your workflow and make your applications more efficient and scalable.

I hope this article has been helpful. Feel free to ask any questions if you need further clarification or additional details.

