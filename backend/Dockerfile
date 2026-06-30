# Stage 1 — build with patched Go dependencies
FROM golang:1.26-alpine AS build
WORKDIR /app

# Copy source code
COPY go.mod go.sum main.go main_test.go ./

# Update vulnerable dependencies during build
RUN go mod edit -require=golang.org/x/crypto@v0.31.0 && \
    go mod edit -require=golang.org/x/net@v0.33.0 && \
    go mod edit -require=golang.org/x/sys@v0.24.0 && \
    go mod edit -require=golang.org/x/text@v0.16.0 && \
    go mod tidy && \
    go mod download

# Build static binary
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o backend .

# Stage 2 — distroless runtime (zero vulnerabilities)
FROM gcr.io/distroless/base-debian12:nonroot
WORKDIR /app
COPY --from=build --chown=nonroot:nonroot /app/backend .
EXPOSE 8080
CMD ["/app/backend"]
