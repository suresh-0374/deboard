# Stage 1 — build app with latest Go
FROM golang:1.25-alpine AS build
WORKDIR /app
COPY go.mod main.go main_test.go ./
# Upgrade all x modules to latest (compatible with Go 1.25)
RUN go get -u golang.org/x/crypto golang.org/x/net golang.org/x/sys golang.org/x/text
RUN go mod tidy
RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o backend .

# Stage 2 — Alpine runtime
FROM alpine:3.20
RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001 -G appgroup
WORKDIR /app
COPY --from=build --chown=appuser:appgroup /app/backend .
USER appuser
EXPOSE 8080
CMD ["/app/backend"]
