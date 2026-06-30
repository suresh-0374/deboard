# Stage 1 — build the app with updated Go runtime
FROM golang:1.26-alpine AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o backend .

# Stage 2 — runtime image using distroless (no shell, minimal attack surface)
FROM gcr.io/distroless/base-debian12:nonroot
WORKDIR /app
COPY --from=build --chown=nonroot:nonroot /app/backend .
EXPOSE 8080
CMD ["/app/backend"]