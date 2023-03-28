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


### Inventory file
Vagrantfile provisions a virtual machine running Ubuntu 20.04, with Ansible as the provisioning tool. We also define an inventory file to specify the hosts that Ansible will manage.
```
[web_servers]
webserver ansible_host=192.168.33.12  ansible_port=2222 ansible_user=vagrant ansible_private_key_file=.vagrant/machines/default/virtualbox/private_key


```
### Vagrantfile
```

Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2204"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    end
end


```

### Playbook.yml
The playbook.yml file defines the tasks that Ansible will execute on the managed hosts. 
```
- name: Configure Application - Clone Repo, Create Vagrant VM, and Set up App in Containers
  hosts: all
  become: true

  vars:
    app_repository: "https://github.com/Emmanuel-9/yolo.git"
    app_branch: "master"
    app_root_dir: "/vagrant/myapp"
    vm_name: "yolovm"
    vm_box: "ubuntu/20.04"
    vm_network_ip: "192.168.56.10"
    role_name: "Emmanuel-9"

  tasks:
    - name: Clone App Repository from GitHub
      git:
        repo: "{{ app_repository }}"
        dest: "{{ app_root_dir }}"
        version: "{{ app_branch }}"

    - name: Install Docker and Docker Compose
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present
    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Update apt and Install Docker-CE
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Build and Run Docker Containers (Client, Backend)
      command: docker compose up -d
      args:
        chdir: "{{ app_root_dir }}"

    - name: Test if App is Accessible from Browser
      uri:
        url: "http://{{ vm_network_ip }}:3000"
        return_content: yes
        status_code: 201
      register: app_response

    - name: Display App Response
      debug:
        var: app_response.content
```
