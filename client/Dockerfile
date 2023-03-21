FROM node:13.12.0
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD [ "npm", "run", "start" ]