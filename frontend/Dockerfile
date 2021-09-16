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
