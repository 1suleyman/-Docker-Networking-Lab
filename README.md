# 🐳 Docker Networking Lab

In this lab, I learned how to **explore Docker networks, inspect network settings, attach containers to specific networks, create custom bridge networks, and deploy a multi-container application (Web App + MySQL) on a user-defined network**.

---

## 📋 Lab Overview

**Goal:**

* List and inspect existing Docker networks
* Identify network IDs and subnets
* Attach containers to default and custom networks
* Create a new user-defined bridge network
* Deploy MySQL and a web application on the same network
* Use environment variables for inter-container communication

**Learning Outcomes:**

* Use `docker network ls` and `docker network inspect`
* Understand default networks: bridge, host, none
* Create networks with custom subnets and gateways
* Attach containers to networks using `--network`
* Link containers using `--link` (legacy, but used in lab)
* Deploy multi-container apps that communicate through networking

---

## 🛠 Step-by-Step Journey

### **Step 1: List Existing Networks**

**Command:**

```bash
docker network ls
```

Found **3 networks**:

* bridge
* host
* none

✔️ Default Docker installation networks

---

### **Step 2: Identify the Bridge Network ID**

Network ID for **bridge**:

```
32… (full ID truncated)
```

✔️ Correctly identified through `docker network ls`

---

### **Step 3: Inspect the Network Attached to alpine-1**

**Command:**

```bash
docker inspect alpine-1
```

Under the `Networks` section:

```
c9a… → host network
```

✔️ alpine-1 is attached to the **host** network.

---

### **Step 4: Identify the Subnet for the Bridge Network**

**Command:**

```bash
docker network inspect bridge
```

Subnet returned:

```
172.12.0.0/24
```

✔️ Correct subnet for the bridge network

---

### **Step 5: Run alpine-2 on the `none` Network**

**Command:**

```bash
docker run --name alpine-2 --network=none alpine
```

✔️ alpine-2 has **no networking stack**
✔️ No DNS, no IP, cannot reach other containers

---

### **Step 6: Create a Custom Bridge Network**

Specifications:

* Name: `wp-mysql-network`
* Subnet: `182.18.0.0/24`
* Gateway: `182.18.0.1`
* Driver: bridge

**Command:**

```bash
docker network create \
  --driver bridge \
  --subnet 182.18.0.0/24 \
  --gateway 182.18.0.1 \
  wp-mysql-network
```

Verified using:

```bash
docker network inspect wp-mysql-network
```

✔️ Custom network created successfully

---

### **Step 7: Deploy MySQL (MySQL 5.7) on the Custom Network**

Container name: `mysql-db`
Required variable: `MYSQL_ROOT_PASSWORD=DB_PASS123`

**Command:**

```bash
docker run -d \
  -e MYSQL_ROOT_PASSWORD=DB_PASS123 \
  --name mysql-db \
  --network wp-mysql-network \
  mysql:5.7
```

✔️ MySQL running
✔️ Accessible only on the custom network

---

### **Step 8: Deploy Web App Connected to MySQL**

Specifications:

* Image: `kodekloud/simple-webapp-mysql`
* Host port: **38080** → Container port **8080**
* Environment variables:

  * `DB_HOST=mysql-db`
  * `DB_PASS=DB_PASS123`
* Must attach to `wp-mysql-network`
* Must also link to MySQL (legacy requirement)

**Command:**

```bash
docker run \
  --network=wp-mysql-network \
  -e DB_HOST=mysql-db \
  -e DB_PASS=DB_PASS123 \
  -p 38080:8080 \
  --name webapp \
  --link mysql-db:mysql-db \
  -d \
  kodekloud/simple-webapp-mysql
```

✔️ Web app deployed
✔️ Communicates with MySQL through the custom network
✔️ Successfully renders in browser

---

### **Step 9: Validate Application**

Clicked on:

```
host:38080
```

Output:

✔️ **“Successfully connected to the MySQL database.”**

✔️ Full multi-container networking works correctly
✔️ End of lab

---

## ✅ Key Commands Summary

| Task                          | Command / Notes                                   |
| ----------------------------- | ------------------------------------------------- |
| List networks                 | `docker network ls`                               |
| Inspect network               | `docker network inspect NETWORK`                  |
| Run container on none network | `docker run --network=none alpine`                |
| Create custom network         | `docker network create --subnet --gateway NAME`   |
| Run MySQL with password       | `docker run -e MYSQL_ROOT_PASSWORD=... mysql:5.7` |
| Attach container to network   | `--network NETWORK_NAME`                          |
| Link containers               | `--link A:B`                                      |
| Expose ports                  | `-p HOST:CONTAINER`                               |

---

## 💡 Notes / Tips

* Default Docker networks: **bridge**, **host**, **none**
* User-defined networks are isolated from default bridge
* Containers on the same user-defined network resolve each other by **name**
* Linking is deprecated, but still works for older labs
* Custom networks allow assigning consistent subnets and gateways
* Web apps communicating with databases MUST be on the same network

---

## 📌 Lab Summary

| Step                   | Status | Key Takeaways                          |
| ---------------------- | ------ | -------------------------------------- |
| List networks          | ✅      | 3 default networks: bridge, host, none |
| Find bridge ID         | ✅      | Found via `docker network ls`          |
| Check alpine-1 network | ✅      | Attached to host network               |
| Get bridge subnet      | ✅      | 172.12.0.0/24                          |
| Run alpine-2 on none   | ✅      | No networking allowed                  |
| Create custom network  | ✅      | `wp-mysql-network` created             |
| Deploy MySQL           | ✅      | `mysql-db` running on custom network   |
| Deploy Web App         | ✅      | Connected to MySQL, exposed on 38080   |
| Validate app           | ✅      | Successfully connected to database     |

---

## ✅ References

* [Docker Networks Overview](https://docs.docker.com/network/)
* [User-Defined Bridge Networks](https://docs.docker.com/network/bridge/)
* [Networking with Docker Containers](https://docs.docker.com/config/containers/container-networking/)
* [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
