FROM tutum/nginx
RUN rm /etc/nginx/sites-enabled/default
ADD sites-enabled/ /etc/nginx/sites-enabled

# Add SSL certificates
ADD ssl/ /etc/nginx/ssl
