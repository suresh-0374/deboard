# Stage 1 — build
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Download deps first for better layer caching.
COPY go.mod go.sum ./
RUN go mod download

# Build the binary.
COPY . .
RUN CGO_ENABLED=0 go build -o devboard-backend .

# Stage 2 — runtime (minimal image)
FROM alpine:3.20

WORKDIR /app

# Copy binary from builder
COPY --from=builder /app/devboard-backend .

# Create non-root user
RUN adduser -D -u 10001 app && chown -R app:app /app
USER app

EXPOSE 8080
ENTRYPOINT ["./devboard-backend"]
