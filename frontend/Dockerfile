# ----Build Stage----
FROM node:24-alpine AS build

RUN apk update && apk upgrade --no-cache

WORKDIR /app

COPY package.json package.json* package-lock.json ./
RUN npm ci && npm cache clean --force

COPY . .
RUN npm run build

# ----Final Stage----
FROM nginx:1.27-alpine AS production

RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

COPY nginx.conf /etc/nginx/conf.d/default.conf

COPY --from=build /app/dist /usr/share/nginx/html

# Set proper permissions for nginx to write logs and cache
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
