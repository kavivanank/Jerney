# --------------------------------------------------
# Build Stage
# --------------------------------------------------
FROM node:26.2-alpine3.22 AS build

WORKDIR /app

# Install security updates
RUN apk update && apk upgrade --no-cache

# Copy dependency manifests
COPY package.json package-lock.json ./

# Install all dependencies deterministically
RUN npm ci

# Copy application source
COPY src/ ./src/

# Remove devDependencies for production
RUN npm prune --omit=dev

# Clean npm cache
RUN npm cache clean --force

# --------------------------------------------------
# Production Stage
# --------------------------------------------------
FROM node:26.2-alpine3.22 AS production

# Install security updates + dumb-init
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache dumb-init=1.2.5-r3

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy only production-ready artifacts
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/src ./src
COPY package.json ./

# Set ownership
RUN chown -R appuser:appgroup /app

# Run as non-root user
USER appuser

EXPOSE 5000

# Proper signal handling
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "src/index.js"]