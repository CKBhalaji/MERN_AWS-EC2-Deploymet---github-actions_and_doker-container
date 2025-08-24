# 🐳 Docker & MongoDB Command Reference for YaliAero Project

This document provides a comprehensive list of Docker and MongoDB commands used during the development and deployment of the **YaliAero** project. These commands help manage the containerized backend environment and perform essential database operations.

---

## 📦 Docker Command Reference

### 1. Basic Docker Commands (Single Container)

| Command                                 | Purpose                                                |
| --------------------------------------- | ------------------------------------------------------ |
| `docker version`                        | Displays Docker client and server version info.        |
| `docker images`                         | Lists locally stored Docker images.                    |
| `docker ps`                             | Lists currently running containers.                    |
| `docker ps -a`                          | Lists all containers (running + stopped).              |
| `docker pull [image_name]`              | Pulls a Docker image from Docker Hub.                  |
| `docker run -d -p 8080:80 [image_name]` | Starts a container with port mapping in detached mode. |
| `docker stop [container_id/name]`       | Stops a running container.                             |
| `docker rm [container_id/name]`         | Removes a stopped container.                           |
| `docker rmi [image_id/name]`            | Deletes a local Docker image.                          |
| `docker logs [container_id/name]`       | Views container logs.                                  |

---

### 2. Docker Compose Commands (Multi-Container)

| Command                                           | Purpose                                          |
| ------------------------------------------------- | ------------------------------------------------ |
| `docker-compose build`                            | Builds images from `docker-compose.yml`.         |
| `docker-compose up -d`                            | Starts containers in background mode.            |
| `docker-compose up --build`                       | Rebuilds and starts containers.                  |
| `docker-compose up -d --force-recreate [service]` | Recreates specific service container.            |
| `docker-compose build --no-cache [service]`       | Rebuilds a service image ignoring cached layers. |

---

### 3. Stop and Clean-up Commands

| Command                  | Purpose                                              |
| ------------------------ | ---------------------------------------------------- |
| `docker-compose down`    | Stops and removes all containers, networks, volumes. |
| `docker-compose down -v` | Also removes named volumes (resets data).            |

---

### 4. Debugging and Inspection Commands

| Command                                    | Purpose                                    |
| ------------------------------------------ | ------------------------------------------ |
| `docker-compose logs [service]`            | Displays logs for a specific service.      |
| `docker exec -it [container_id/name] bash` | Opens a bash shell in a running container. |

---

## 🍃 MongoDB Shell (`mongosh`) Quick Reference

All commands below are intended to be run inside the MongoDB container via `mongosh`.

---

### 1. Access MongoDB Container

```bash
docker exec -it yaliaero-mongodb bash
```

### 2. Connect to MongoDB via `mongosh`

```bash
mongosh "mongodb://adminUser:${MONGO_PASSWORD}@localhost:27017/yaliaero_db?authSource=admin"
```

---

## 📁 Database & Collection Basics

### 📌 Switch Database

```javascript
use yaliaero_db
```

### 📌 Show All Databases

```javascript
show dbs
```

### 📌 Show Collections

```javascript
show collections
```

### 📌 Create a Collection

```javascript
db.createCollection("users")
```

---

## 👤 User Management Examples

### ➕ Insert a User

```javascript
db.users.insertOne({
  email: 'softwareteam@yaliaero.com',
  password: 'yaliaeroone',
  authorizedPersonName: 'Software Team',
  authorizedPersonMobile: '8976543567',
  organizationName: 'Yaliaerospace',
  role: 'admin',
  isEmailVerified: true,
  isVerified: true,
  createdAt: new Date()
});
```

### 🔍 Find One User

```javascript
db.users.findOne({ email: 'testuser@example.com' });
```

### 🔍 Find All Users

```javascript
db.users.find({});
```

### 🔍 Filter by Role

```javascript
db.users.find({ role: 'admin' });
```

### ✏️ Update a User’s Role & Status

```javascript
db.users.updateOne(
  { email: 'testuser@example.com' },
  {
    $set: {
      role: 'associate',
      isEmailVerified: true,
      isVerified: true
    }
  }
);
```

### ❌ Delete User by Email

```javascript
db.users.deleteOne({ email: 'testuser@example.com' });
```

---

## 📊 Query Enhancements

### 🔽 Sort Users by Creation Date (Newest First)

```javascript
db.users.find({}).sort({ createdAt: -1 });
```

### 🔢 Count Total Users

```javascript
db.users.countDocuments();
```

### 📌 Pagination (Skip & Limit)

```javascript
db.users.find({}).skip(10).limit(5);
```

### 🔍 Projection (Only Email and Role)

```javascript
db.users.find({}, { email: 1, role: 1, _id: 0 });
```

---

## 🔄 Aggregation Examples

### 👥 Group Users by Role

```javascript
db.users.aggregate([
  { $group: { _id: "$role", count: { $sum: 1 } } }
]);
```

### 🧮 Count Verified Users

```javascript
db.users.countDocuments({ isVerified: true });
```

---

## ⚙️ Indexing (Performance)

### ➕ Create Index on Email

```javascript
db.users.createIndex({ email: 1 }, { unique: true });
```

### 🔍 View Indexes

```javascript
db.users.getIndexes();
```

---

## ❓ FAQ Collection Example

### Insert Multiple FAQ Entries

```javascript
db.faqs.insertMany([
  {
    question: "How do I register a new user account?",
    answer: "Click the 'Register' link on the login page and fill in your details. You may need to verify your email before logging in.",
    createdAt: new Date("2024-07-20T10:00:00Z")
  },
  {
    question: "How do I upload and review a flight log?",
    answer: "Go to the 'Flights' section, click 'Upload Log', and select your ULG file.",
    createdAt: new Date("2024-07-20T10:25:00Z")
  }
]);
```

---

## 🛠️ Additional Notes

### 🔗 Map Custom Domain (`your_site.com`) to Localhost

To access Docker service via a domain locally:

**Edit `/etc/hosts` (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows):**

```bash
127.0.0.1 your_site.com
```

Then ensure your app binds to:

```js
app.listen(3000, '0.0.0.0');
```

---

## 🧼 Cleanup & Maintenance

### 🧹 Prune Docker Resources

```bash
docker system prune -a
```

### 🧼 Remove All Stopped Containers

```bash
docker container prune
```

---

## 🔐 Security Best Practices

* Never store passwords in plain text.
* Always hash passwords using `bcrypt` or similar libraries.
* Use environment variables to store sensitive config.

---

## 📚 References

* [Docker Docs](https://docs.docker.com/)
* [MongoDB Shell Docs](https://www.mongodb.com/docs/mongodb-shell/)
* [MongoDB Query Operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
* [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)

---
