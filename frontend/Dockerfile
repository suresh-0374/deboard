# Stage 1 — build the app into static files
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2 — small image that only serves the build
FROM node:20-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Install vite only
RUN npm install -g vite@5 && npm cache clean --force

# Copy built files from build stage
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs vite.preview.config.js ./vite.config.js

# Switch to non-root user
USER nodejs

EXPOSE 4173
CMD ["vite", "preview", "--host", "0.0.0.0", "--port", "4173"]
