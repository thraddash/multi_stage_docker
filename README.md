# docker-compose with Docker multi stage build 
#### Docker images used:  
- node:16-alpine 113MB  
- nginx:alpine 22.9MB  

## 1. Create React App
npm create react-app frontend  
cd frontend  
npm install  
npm start

## 2. Create Dockerfile (use node:16-alpine size: 113 MB)  
#### (after nodejs is installed, docker frontend image size: 443MB)
<details>
  <summary>Dockerfile</summary>
  
```
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# Install app dependencies
# Copies package.json and package-lock.json to Docker environment
COPY package*.json ./

# Install all node packages
RUN npm Install

# Copy everything over to Docker environment
COPY . ./
EXPOSE 3000

# start app
CMD ["npm", "start"] 
```
</details>

## 3. Convert Dockerfile to multi stage build
#### stage 1: create react app on node:16-alpine, RUN npm run build , this will create a build folder for production
#### stage 2: build nginx:alpine size: 22.9MB, Copy static build folder from stage 1: to nginx WORKDIR /usr/share/nginx/html
<details>
  <summary>Updated Dockerfile to multi stage build</summary>
  
```
FROM node:16-alpine as builder

# Set working directory
WORKDIR /app

# Add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# Install app dependencies
# Copies package.json and package-lock.json to Docker environment
COPY package*.json ./

# Install all node packages
RUN npm install

# Copy everything over to Docker environment
COPY . ./
RUN npm run build

# Stage 2 
################################
# pull nginx:alpine 9.82MB
FROM nginx:alpine

# Copy React to container directory
# Set working directory to nginx resource directory
WORKDIR /usr/share/nginx/html

# Remove default nginx static resource
RUN rm -rf ./*

# Copy static build directory from builder stage
COPY --from=builder /app/build .

# Make port 80 availabe to world outside this container
EXPOSE 80

# Run Nginx with global directives and deamon off
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
</details>

## 4. Create docker-compose.yml file
```
version: '3.8'

services:
  frontend:
    image: frontend
    ports:
      - "80:80"
    build:
      dockerfile: Dockerfile
      context: ./frontend
```
docker-compose build  
docker-compose up  
docker-compose down  

<b>React App (production build folder in Nginx size: 23.5MB)</b>
```
REPOSITORY   TAG             IMAGE ID       CREATED             SIZE
frontend     latest          ad777ae9f45f   47 seconds ago      23.5MB
node         16-alpine       183a5b519074   2 days ago          113MB
nginx        alpine          513f9a9d8748   6 days ago          22.9MB
```
<b>React App (development build with node_modules size: 443.5MB)</b>
```
REPOSITORY   TAG             IMAGE ID       CREATED             SIZE
frontend     latest          57fafb37b5f3   46 minutes ago      443.5MB
node         16-alpine       183a5b519074   2 days ago          113MB
nginx        alpine          513f9a9d8748   6 days ago          22.9MB

```
 
 ## 5. (TBA) create backend express server, test frontend to backend with axios
 - test api CRUD using axios
 
 ## 6. (create mongo db? connect to backend server, test)
