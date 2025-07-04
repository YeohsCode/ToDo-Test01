# Use Node.js as the base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY ../../frontend/package*.json ./
RUN npm install

# Copy the rest of the application
COPY ../../frontend/ ./

# Build the Vue.js application
RUN npm run build

# Expose the port the app runs on
EXPOSE 8080

# Command to run the application
CMD ["npm", "run", "serve"]
