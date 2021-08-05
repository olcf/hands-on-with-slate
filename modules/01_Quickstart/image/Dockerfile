FROM nginx:latest

RUN chgrp 0 /var/cache/nginx && chmod 775 /var/cache/nginx

ADD nginx.conf /etc/nginx/nginx.conf
ADD index.html /usr/share/nginx/html
