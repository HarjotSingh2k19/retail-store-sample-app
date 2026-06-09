# Stage 1 — Build
# Need CGO for sqlite3 — use full golang image not alpine
FROM golang:1.23 AS builder
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
# CGO_ENABLED=1 — required for go-sqlite3
RUN CGO_ENABLED=1 GOOS=linux go build -o catalog .

# Stage 2 — Run
# Use debian-slim not alpine — CGO binary needs glibc
FROM debian:bookworm-slim
WORKDIR /app

# Install sqlite3 runtime + wget for healthcheck
RUN apt-get update && apt-get install -y \
    wget \
    libsqlite3-0 \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/catalog .
COPY --from=builder /app/repository/products.json ./repository/products.json
COPY --from=builder /app/repository/tags.json ./repository/tags.json

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:8080/catalog/size || exit 1

ENTRYPOINT ["./catalog"]
