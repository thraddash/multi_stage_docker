FROM node:16-alpine 

#Set working directory
WORKDIR /app

#Add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

#Install app dependencies
#Copies package.json and package-lock.json to Docker environment
COPY package*.json ./

#Install all node packages
RUN npm install

#Copy everything over to Docker environment
COPY . ./
EXPOSE 3000

#Start app
CMD ["npm", "start"]
