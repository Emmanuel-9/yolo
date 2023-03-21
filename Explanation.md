### Backend Dockerfile
The backend Dockerfile defines the image for the server-side application. Here's a breakdown of what the file does:
```
* FROM node:14: This line specifies the base image for the Docker image, in this case, the official  Node.js 14 image from Docker Hub.
* WORKDIR /app: This sets the working directory for the image to /app.
* COPY package*.json ./: This line copies the package.json and package-lock.json files to the /app directory in the Docker image.
* RUN npm install: This line installs the Node.js dependencies for the server-side application.
* COPY . .: This line copies the entire contents of the current directory (the server-side application code) to the /app directory in the Docker image.
* EXPOSE 5000: This line exposes port 5000 for the server-side application.
* CMD ["npm", "start"]: This line specifies the command that should be run when the Docker image is started, in this case, running the npm start command to start the server-side application.
```
### Client Dockerfile
The client Dockerfile defines the image for the client-side application. Here's a breakdown of what the file does:
```
* FROM node:14: This line specifies the base image for the Docker image, in this case, the official Node.js 14 image from Docker Hub.
* WORKDIR /app: This sets the working directory for the image to /app.
* COPY package*.json ./: This line copies the package.json and package-lock.json files to the /app directory in the Docker image.
* RUN npm install: This line installs the Node.js dependencies for the client-side application.
* COPY . .: This line copies the entire contents of the current directory (the client-side application code) to the /app directory in the Docker image.
* EXPOSE 3000: This line exposes port 3000 for the client-side application.
* CMD ["npm", "start"]: This line specifies the command that should be run when the Docker image is started, in this case, running the npm start command to start the client-side application.

```

### Docker compose file
In this Docker Compose file, we define three services: frontend, backend, and mongo. The frontend and backend services are based on the custom images while the mongo service uses the official mongo image.
```
version: '3.9'

services:
  backend:
    container_name: yolo-backend
    build: ./backend
    ports:
      - "5000:5000"
    env_file:
      - ./backend/.env
    networks:
      - yolo-network
    depends_on:
      - db  
  
  client:
    container_name: yolo-frontend
    build: ./client 
    ports:
      - "3000:3000"
    networks:
      - yolo-client
    depends_on:
      - backend

  db:
      image: mongo
      ports:
        - 27017:27017
      container_name: mongo-db
      networks:
        - yolo-client
      volumes:
        - mongo-data:/data/db      

networks:
  yolo-client:
    driver: bridge

volumes:
  mongo-data:
    driver: local    
```


