# Stage 1 — Build
# Need full Node with devDependencies to compile TypeScript
FROM node:20-alpine AS builder
WORKDIR /app

# Install dependencies first (cached layer)
COPY package*.json yarn.lock ./
RUN yarn install --frozen-lockfile

# Copy source and compile TypeScript → JavaScript
COPY . .
RUN yarn build

# Stage 2 — Run
# Fresh Node image, only copy compiled output
FROM node:20-alpine
WORKDIR /app

# Only install production dependencies
COPY package*.json yarn.lock ./
RUN yarn install --frozen-lockfile --production

# Copy compiled JavaScript from Stage 1
COPY --from=builder /app/dist ./dist

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8080/health || exit 1

# serve:prod from package.json = node dist/main
CMD ["node", "dist/main"]