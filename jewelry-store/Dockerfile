# Use Node.js LTS for React
FROM node:18

# Set working directory
WORKDIR /app

# Copy package.json and lock files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy frontend code
COPY . .

# Expose port for development server
EXPOSE 3000

# Start React development server
CMD ["npm", "start"]
