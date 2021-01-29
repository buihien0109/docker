#stage 1 ======================

FROM node:13-alpine as builder

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

RUN npm run build

#stage 2 ======================
FROM nginx:1.17-alpine

COPY --from=builder /app/dist /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]
