FROM nginx:1.21.6

RUN apt-get update
RUN apt-get install -y openssl

COPY ./code/entrypoint.sh /entrypoint.sh
COPY ./code/default.conf /etc/nginx/conf.d/default.conf

CMD ["/bin/bash", "/entrypoint.sh"]
