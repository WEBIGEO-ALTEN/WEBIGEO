# Use an official Nginx runtime as the base image
FROM node:latest

# Install Node.js and npm
RUN apt update && apt install -y git nodejs npm
RUN npm install -g serve
RUN git clone https://github.com/WEBIGEO-ALTEN/WEBIGEO_FRONT.git
WORKDIR WEBIGEO_FRONT/
RUN git pull
RUN npm i
RUN npm run build

CMD ["ls"]
CMD ["serve", "-s", "build", "-l", "3000"]
