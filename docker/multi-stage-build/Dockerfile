# syntax=docker/dockerfile:1

# === Builder
FROM golang:1.23.4 AS builder

WORKDIR /app

# go.modとをコピー
COPY go.mod go.sum ./

# ソースコードをコピー
COPY sample.go sample.go

RUN go build -o main .

# === Runner
FROM busybox AS runner

WORKDIR /app

RUN adduser \
  --disabled-password \
  --gecos "" \
  --home "/nonexistent" \
  --shell "/sbin/nologin" \
  --no-create-home \
  --uid "1001" \
  app

USER app

COPY --from=builder --chown=app:app /app/main /bin/main

EXPOSE 8080

CMD ["main"]
