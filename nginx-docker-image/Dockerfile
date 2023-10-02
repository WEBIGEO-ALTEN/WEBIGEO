# Use an official Nginx runtime as the base image
FROM node:latest

#Env variables
ARG API_DNS=api.webigeo-pre.dcpepper.cloudns.ph

# Install Node.js and npm
RUN apt update && apt install -y git nodejs npm
RUN npm install -g serve

#Clone repository
RUN git clone https://github.com/WEBIGEO-ALTEN/WEBIGEO_FRONT.git
WORKDIR WEBIGEO_FRONT/

#Create the .env file depending on the environment
RUN echo "REACT_APP_API_DNS=$API_DNS" > .env

#Install all dependencies
RUN npm i
RUN npm run build

#Expose serve on port 3000
CMD ["serve", "-s", "build", "-l", "3000"]
