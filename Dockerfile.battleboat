FROM ubuntu
MAINTAINER julien
RUN apt-get update
RUN apt-get install -y nginx
COPY ./battleboat/ /var/www/html
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
EXPOSE 80

