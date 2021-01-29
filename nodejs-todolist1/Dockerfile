FROM node:13-alpine

WORKDIR /app

COPY package.json .

COPY package-lock.json .

RUN npm install

COPY . .

CMD ["node", "server.js"]