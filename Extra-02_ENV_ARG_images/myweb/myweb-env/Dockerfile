

FROM nginx:latest
WORKDIR /usr/share/nginx/html
ENV COLOR="red"
RUN apt update ; apt install curl -y
COPY . .
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 CMD curl -f http://localhost/ || exit 1
CMD sed -e s/COLOR/"$COLOR"/ myweb.html > index.html ; rm myweb.html ; nginx -g 'daemon off;'