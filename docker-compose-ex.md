## Task 5: Viết docker- compose.yml cho ứng dụng NodeJS + PostgreSQL
1. Viết Dockerfile để tạo Docker Image cho ứng dụng NodeJS:
https://github.com/handuy/nodejs-todolist
2. Viết docker-compose.yml để triển khai ứng dung

### Task 1: Viết DockerFile cho nodejs
- Sử dụng base image là **node:13-alpine**
- Câu lệnh để start ứng dụng: **node server.js**
- Đóng gói thành Docker Image bằng lệnh docker build
- Mặc định port container : **8080**
```
FROM node:13-alpine

WORKDIR /app

COPY package.json .

COPY package-lock.json .

// Cách viết khác:
// COPY package-*.json . (Cách 1)
// COPY package.json package-lock.json . (Cách 2)

RUN npm install

COPY . .

CMD ["node", "server.js"]
```

### Viết docker-compose.yml

1. Với container chạy NodeJS:
    - Sử dụng Docker Image cho ứng dụng NodeJS vừa build ở bước 1
    - Expose cổng **8080** của NodeJS app ra cổng **8181** của máy host
    - PostgreSQL được khởi tạo trước, sau đó mới đến NodeJS app

2. Với container chạy PostgreSQL
    - Mount volume file **init.sql** từ host vào **/docker-entrypoint-initdb.d/init.sql** của
PostgreSQL container để tạo sẵn bảng
    - Mount volume thư mục **/var/lib/postgresql/data** của container ra một thư mục bất kỳ
trên máy host
    - Tên của service chạy PostgreSQL phải là **db**
    - Biến môi trường **POSTGRES_PASSWORD** có giá trị là **postgres**
```
version: '2.2'
services: 
    db :
        image: postgres:alpine
        restart: always
        volumes: 
            - /Users/techmaster/Desktop/init.sql:/docker-entrypoint-initdb.d/init.sql
            - /Users/techmaster/Desktop/data:/var/lib/postgresql/data
        environment: 
            POSTGRES_PASSWORD : postgres
    
    web :
        image: nodejs-app:v1
        depends_on: 
            - db
        ports: 
            - "8181:8080"
        restart: always
        
```

---

## Task 6: Viết docker- compose.yml cho ứng dụng SpringBoot + MySQL

### - Sử dụng base image là maven
### - Câu lệnh để start ứng dụng: mvn spring-boot:run
### - Build thành Docker Image bằng lệnh docker build

1. Với container chạy MySQL:
    - Mount volume file **obo.sql** từ host vào **/docker-entrypoint-initdb.d/init.sql** của MySQL container để
mockup data.
    - Link download file obo.sql: https://techmaster.vn/media/download/source-code/btq4ftc51co41h2qcrc0
    - Tên của service chạy MySQL phải là **mysql**
    - Khi container chạy MySQL được khởi tạo, bên trong đã có sẵn 1 database tên là **obo**, 1 user **admin** với
password là **123456** (tham khảo cách set biến môi trường cho MySQL tại https://hub.docker.com/_/mysql )
2. Với container chạy ứng dụng SpringBoot
    - Sử dụng Docker Image vừa build ở bước trước
    - Expose cổng **8080** của SpringBoot app ra cổng **8005** của máy host
    - MySQL được khởi tạo trước, sau đó mới đến SpringBoot app

```
version: '2.2'
services: 
    mysql :
        image: mysql
        restart: always
        volumes: 
            - /Users/techmaster/Desktop/obo.sql:/docker-entrypoint-initdb.d/init.sql
        environment: 
            MYSQL_ROOT_PASSWORD : mysql 
            MYSQL_DATABASE : obo 
            MYSQL_USER : admin 
            MYSQL_PASSWORD : 123456
    
    web :
        image: project-maven:v1
        depends_on: 
            - mysql
        ports: 
            - "8005:8080"
        restart: always
```

---

## Task 7: Viết docker- compose.yml cho ứng dụng NodeJS + MongoDB

### Yêu cầu 1: Viết Dockerfile
- Dockerfile chạy trên base image **node:12-alpine**

- Build ra 1 Docker Image có tên là **demo-service**

### Yêu cầu 2: Viết file docker-compose.yml
- File docker-compose.yml gồm 2 service có tên là **mongodb** và **nodejs**

1. Cấu hình service mongodb:

    - Sử dụng image **mongo:latest**

    - Mount volume thư mục **/data/db** ra 1 thư mục bên ngoài host

```
FROM node:12-alpine

WORKDIR /app

COPY package.json .

COPY package-lock.json .

RUN npm install

COPY . .

CMD ["npm", "start"]
```

2. Cấu hình service nodejs:

    - Chạy sau service mongodb

    - Sử dụng Docker Image demo-service vừa build ở yêu cầu 1

    - Expose cổng **3000** của ứng dụng ra cổng **8000** của ngoài host

    - Định nghĩa 2 biến môi trường:                   
        - **MONGODB_URI=mongodb://mongodb:27017/demo**
        - **PORT=3000**

```
version: '2.2'
services: 
    mongodb :
        image: mongo:latest
        restart: always
        volumes: 
            - /Users/techmaster/Documents/data:/data/db
    
    nodejs :
        image: demo-service:latest
        depends_on: 
            - mongodb
        ports: 
            - "8080:3000"
        restart: always
        environment: 
            MONGODB_URI : mongodb://mongodb:27017/demo
            PORT : 3000
```
