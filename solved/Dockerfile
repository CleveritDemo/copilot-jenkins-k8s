# Base image
FROM node:14-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json /app/
RUN npm install

# Copy the rest of the app files
COPY . /app

# Expose port 3000
EXPOSE 3000

# Command to start the app
CMD ["node", "app.js"]