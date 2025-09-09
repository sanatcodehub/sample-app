FROM node:18-alpine as build

WORKDIR /app

# Copy package files first for caching
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Build the React app
RUN npm run build

# ---------------------
# Production stage
# ---------------------
FROM nginx:alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built React files to nginx html folder
COPY --from=build /app/build /usr/share/nginx/html

# Ensure permissions
RUN mkdir -p /var/cache/nginx /var/run/nginx \
    && chown -R nginx:nginx /usr/share/nginx/html /var/cache/nginx /var/run/nginx

# ⚠️ Do NOT switch to USER nginx here
# nginx official image already handles user switching safely

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost/ || exit 1

# Start nginx in foreground
CMD ["nginx", "-g", "daemon off;"]

