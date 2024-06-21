# Stage 1: Build the smart_contract
FROM node:16 as smart_contract-build

# Set the working directory inside the container
WORKDIR /app/smart_contract

# Copy the smart_contract package.json and package-lock.json files
COPY smart_contract/package*.json ./

# Install smart_contract dependencies
RUN npm install

# Copy the rest of the smart_contract code
COPY smart_contract .

# Stage 2: Build the client
FROM node:16 as client-build

# Set the working directory inside the container
WORKDIR /app/client

# Copy the client package.json and package-lock.json files
COPY client/package*.json ./

# Install client dependencies
RUN npm install

# Copy the rest of the client code
COPY client .

# Build the client
RUN npm run build

# Stage 3: Serve the client application with Nginx
FROM nginx:alpine

# Copy the client build output from Stage 2 to Nginx's html directory
COPY --from=client-build /app/client/build /usr/share/nginx/html

# Copy custom Nginx configuration if necessary
# COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80 to the outside world
EXPOSE 80

# Start Nginx when the container launches
CMD ["nginx", "-g", "daemon off;"]
