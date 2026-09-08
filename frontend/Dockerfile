# syntax=docker/dockerfile:1
# --- Build stage: cargo + trunk → a directory of static files ---
FROM rust:1-bookworm AS builder
WORKDIR /src
RUN rustup target add wasm32-unknown-unknown \
    && cargo install --locked trunk
COPY . .
RUN trunk build --release

# --- Runtime: unprivileged nginx (non-root uid 101, listens on 8080) ---
FROM nginxinc/nginx-unprivileged:1.27-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /src/dist /usr/share/nginx/html
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -q --spider http://localhost:8080/healthz || exit 1
CMD ["nginx", "-g", "daemon off;"]
