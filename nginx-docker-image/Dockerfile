# Use a base image with Ubuntu 20.04
FROM ubuntu:20.04

# Update the package lists and install Nginx
RUN apt-get update && apt-get install -y nginx

# Expose ports for Nginx (80 and 443)
EXPOSE 80
EXPOSE 443

# Install curl to fetch additional software if needed
RUN apt-get update && apt-get install -y curl

# Set an environment variable
ENV MY_ENV_VARIABLE=value

# Set the working directory to /app (you can change this to your preference)
WORKDIR /app

# Copy your application files to the container (if you have any)
COPY . /app/

# Start Nginx (and your application if you have one)
CMD ["nginx", "-g", "daemon off;"]
