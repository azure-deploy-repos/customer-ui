
# =====================================================
# Stage 1: Build arguments
# =====================================================

ARG NODE_VERSION=20
ARG APP_NAME=booktrack-customerui
ARG APP_VERSION=1.0.0

# =====================================================
# Stage 2: Build React/Vite application
# =====================================================

FROM node:${NODE_VERSION}-alpine AS builder

ARG APP_NAME
ARG APP_VERSION

ARG VITE_AUTH_API_URL
ARG VITE_PRODUCT_API_URL
ARG VITE_ORDER_API_URL
ARG VITE_TRACKING_API_URL
ARG VITE_USER_API_URL

LABEL application="${APP_NAME}" \
      version="${APP_VERSION}"

WORKDIR /app

# Vite configuration: values are embedded during npm run build
ENV VITE_APP_NAME=${APP_NAME} \
    VITE_APP_VERSION=${APP_VERSION} \
    VITE_AUTH_API_URL=${VITE_AUTH_API_URL} \
    VITE_PRODUCT_API_URL=${VITE_PRODUCT_API_URL} \
    VITE_ORDER_API_URL=${VITE_ORDER_API_URL} \
    VITE_TRACKING_API_URL=${VITE_TRACKING_API_URL} \
    VITE_USER_API_URL=${VITE_USER_API_URL}

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

# =====================================================
# Stage 3: Production Nginx runtime
# =====================================================

FROM nginxinc/nginx-unprivileged:stable-alpine AS runtime

ARG APP_NAME=booktrack-customerui
ARG APP_VERSION=1.0.0

LABEL application="${APP_NAME}" \
      version="${APP_VERSION}" \
      security.non-root="true"

# Copy only the compiled frontend files
COPY --from=builder --chown=101:101 \
    /app/dist /usr/share/nginx/html

# Copy Nginx configuration
COPY --chown=101:101 nginx.conf \
    /etc/nginx/conf.d/default.conf

# Run as non-root
USER 101:101

EXPOSE 8080

# Container healthcheck
HEALTHCHECK --interval=30s \
            --timeout=5s \
            --start-period=10s \
            --retries=3 \
    CMD wget -q --spider http://127.0.0.1:8080/ || exit 1

# Exec-form command: no shell is used to start Nginx
CMD ["nginx", "-g", "daemon off;"]
