FROM nginx:1.13.12-alpine

COPY keys /etc/nginx/keys
COPY static /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/tools.monday.health.conf

EXPOSE 443
