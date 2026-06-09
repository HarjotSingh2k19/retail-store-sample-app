# Stage 1 — Build
# Go compiler to build a single static binary
FROM golang:1.23-alpine AS builder
WORKDIR /app

# Download dependencies first (cached layer)
COPY go.mod go.sum ./
RUN go mod download

# Copy source and build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o catalog .

# Stage 2 — Run
# alpine is tiny — Go binary needs almost no runtime dependencies
FROM alpine:3.19
WORKDIR /app

# Install wget for health check
RUN apk --no-cache add wget

# Copy only the compiled binary from Stage 1
COPY --from=builder /app/catalog .

# Copy static data files the app needs
COPY --from=builder /app/repository/products.json ./repository/products.json
COPY --from=builder /app/repository/tags.json ./repository/tags.json

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8080/catalogue/size || exit 1

ENTRYPOINT ["./catalog"]