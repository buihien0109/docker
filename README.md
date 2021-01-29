## Network

- Tạo network
```
docker network create [name-network]
```

- Đặt container vào trong network
```
docker network connect [name-network][container-ID]
```

- Đặt container vào trong network lúc run container
```
docker run -d --network node-todo --name my-node -p 8181:8080 handuy1992/node-todo:latest
```

---

## Bài tập : Ứng dụng todolist Node

1. Tạo network
```
docker network create my-node
```

2. Tạo container db
// https://hub.docker.com/_/postgres

```
docker run -d --name db -e POSTGRES_PASSWORD=postgres -e PGDATA=/var/lib/postgresql/data/pgdata -v /var/lib/postgresql/data --network node-todo handuy1992/postgres-12
```

3. Tạo container node
```
docker run -d --network node-todo --name my-node -p 8181:8080 handuy1992/node-todo:latest
```

---

## Bài tập : Ứng dụng OBO Java SpringBoot

1. Tạo network
```
docket network create spring-app
```

2. Chạy container mysql
```
docker run -d --name mysql --network spring-app -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=mysql -e MYSQL_DATABASE=obo -e MYSQL_USER=admin -e MYSQL_PASSWORD=123456 handuy1992/mysql
```

3. Chạy container OBO
```
docker run -d --name obo --network spring-app -p 8005:8080 handuy1992/obo:latest
```

---

## Build image
```
docker build -t calculator:v1 .

// Trường hợp đặt tên docker file không đúng chuẩn
// Ví dụ: Dockerfile.txt
docker build -t calculator:v1 -f Dockerfile.txt
```

---

## Task 1: Viết Dockerfile đóng gói ứng dụng NodeJS
Cho source code của 1 ứng dụng React tại địa chỉ: https://github.com/ahfarmer/calculator
### Yêu cầu:
- Clone source code về máy
- Bên trong thư mục chứa source code, viết file Dockerfile
- Build ra Docker image
- Chạy thử Docker image, expose ra cổng 8080 trên máy host
```
FROM node:alpine

WORKDIR /app

COPY package.json .

COPY package-lock.json .

RUN npm install

COPY . .

CMD ["npm", "run", "start"]
```

---

## Task 2: Viết Dockerfile đóng gói ứng dụng Java

1. Tạo 1 thư mục có tên **docker-java** để chứa source code

2. Bên trong thư mục docker-java: Tạo file **Hello.java** (nội dung chi tiết xem ở slide
dưới)

3. Bên trong thư mục docker-java: Tạo file Dockerfile (yêu cầu chi tiết xem ở slide
dưới)

4. Dùng lệnh docker build để tạo Docker Image

5. Tạo container từ Image vừa tạo bằng lệnh: docker run tên-image-vừa-tạo. Màn hình terminal cần hiển thị dòng chữ: Chào các bạn

```
FROM openjdk:8-alpine

WORKDIR /app

COPY . .

RUN javac Hello.java

CMD ["java", "Hello"]
```

---

## Task 3: Viết Dockerfile cho ứng dụng Angular

### Source code của ứng dụng:
https://github.com/handuy/angular-hero

### Viết Dockerfile dưới dạng multi-stage, sau đó build ứng dụng thành Docker Image và khởi tạo container

1. Clone source code về máy
2. Bên trong thư mục chứa source code, tạo file Dockerfile, chia làm 2 stage:
- Stage 1:
    - Sử dụng base image node:13-alpine
    - Set WORKDIR /app
    - Copy file package.json vào image, sau đó chạy lệnh npm install
    - Copy các file và thư mục còn lại vào image, sau đó chạy lệnh npm run build

- Stage 2:
    - Sử dụng base image nginx:1.17-alpine
    - Copy thư mục /app/dist được tạo ra từ stage 1 vào thư mục /usr/share/nginx/html
    - Dùng lệnh sau để start container: nginx -g daemon off;

```
# stage 1 ======================

FROM node:13-alpine as builder

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

RUN npm run build

# stage 2 ======================

FROM nginx:1.17-alpine

COPY --from=builder /app/dist /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"] # (Có thể bỏ)

```


---

## Task 4: Viết Dockerfile đóng gói ứng dụng SpringBoot

### - Source code của ứng dụng:
https://github.com/handuy/spring-app-demo
### - Viết Dockerfile dưới dạng multi-stage, sau đó build ứng dụng thành Docker Image và khởi tạo container


1. Clone source code về máy
2. Bên trong thư mục chứa source code, tạo file Dockerfile, chia làm 2 stage:

- Stage 1:
    - Sử dụng base image **maven:ibmjava-alpine**
    - Copy source code vào image
    - Từ source code build ra file **websocket-demo-0.0.1-SNAPSHOT.jar** bằng lệnh:
    **mvn clean package.**
    - File **websocket-demo-0.0.1-SNAPSHOT.jar** sẽ nằm trong thư mục **target** của source code
- Stage 2:
    - Sử dụng base image **openjdk:8-alpine**
    - Copy file **websocket-demo-0.0.1-SNAPSHOT.jar** từ stage 1 sang stage 2
    - Dùng lệnh sau để start container:
    **java -Djava.security.egd=file:/dev/./urandom -jar websocket-demo-0.0.1-SNAPSHOT.jar**

3. Từ Dockerfile build thành Docker Image có tên là spring-app
4. Khởi động container từ image spring-app: container chạy ngầm,
expose cổng 8080 của container ra cổng 9002 của host
5. Truy cập localhost:9002 để kiểm tra kết quả

```

#Stage 1 =========================================

FROM maven:ibmjava-alpine as builder

COPY . /app

WORKDIR /app

RUN mvn clean package

#Stage 2 =========================================

FROM openjdk:8-alpine

WORKDIR /java

COPY --from=builder /app/target/websocket-demo-0.0.1-SNAPSHOT.jar /java

CMD ["java", "-Djava.security.egd=file:/dev/./urandom ", "-jar", "websocket-demo-0.0.1-SNAPSHOT.jar"]
```

---