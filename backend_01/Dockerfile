# ---- Build stage ----
    # Notice we used older image here for our project because we want to catch security issues in our devsecops project. 
    # We will be using the latest image for our production stage to ensure we have the latest security patches.
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --only=production && npm cache clean --force

# ---- Production stage ----
FROM node:20-alpine AS production

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Security: remove unnecessary packages
# Intentionally did not specify version here becuase we want to catch security issues in dumb-init for our devsecops project
RUN apk --no-cache add dumb-init 

WORKDIR /app

# Copy only production dependencies from build stage
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: own files by non-root user
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 5000

# Security: use dumb-init for proper signal handling (PID 1 problem)
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/index.js"]
