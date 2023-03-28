FROM orega/yolo:1.1
WORKDIR /usr/app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 5000
CMD [ "npm", "run","start" ]
