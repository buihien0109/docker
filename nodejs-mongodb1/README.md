## Yêu cầu triển khai ứng dụng NodeJS + MongoDB

### Yêu cầu 1: Viết Dockerfile

Dockerfile chạy trên base image **node:12-alpine**

Build ra 1 Docker Image có tên là **demo-service**

### Yêu cầu 2: Viết file docker-compose.yml

File docker-compose.yml gồm 2 service có tên là **mongodb** và **nodejs**

Cấu hình service mongodb:

- Sử dụng image **mongo:latest**

- Mount volume thư mục **/data/db** ra 1 thư mục bên ngoài host

Cấu hình service nodejs:

- Chạy sau service mongodb

- Sử dụng Docker Image **demo-service** vừa build ở yêu cầu 1

- Expose cổng 3000 của ứng dụng ra cổng 8000 của ngoài host

- Định nghĩa 2 biến môi trường: MONGODB_URI=mongodb://mongodb:27017/demo, PORT=3000