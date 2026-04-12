# ── Stage 1: Build ────────────────────────────────────────────────────────────
FROM golang:1.23-alpine AS builder

# git: required by go mod for VCS-stamped modules
RUN apk add --no-cache git ca-certificates tzdata

WORKDIR /app

# Copy module manifests first so Docker caches the dependency layer separately.
# go.sum* — the asterisk makes go.sum optional on a fresh clone;
# go mod tidy generates it on first build.
COPY go.mod go.sum* ./
RUN go mod tidy

# Copy remaining source and compile a fully static binary.
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build \
    -ldflags="-s -w -extldflags=-static" \
    -trimpath \
    -o /bin/api \
    ./cmd/api

# ── Stage 2: Runtime ──────────────────────────────────────────────────────────
# alpine gives us a shell + wget so Docker's CMD-SHELL health check works.
# It also keeps the image small (~12 MB vs ~300 MB for the full Go base image).
FROM alpine:3.20 AS runtime

RUN apk add --no-cache ca-certificates tzdata wget

# Run as non-root for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Migrations are embedded inside the binary via embed.FS — no volume needed.
COPY --from=builder /bin/api /api

EXPOSE 4000

ENTRYPOINT ["/api"]
