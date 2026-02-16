FROM ubuntu
WORKDIR /app
RUN apt-get update -y
RUN apt-get install python3 -y
RUN apt-get install python3-pip -y
RUN pip3 install Flask --break-system-packages
COPY . .
EXPOSE 80
CMD ["python3", "./welcome.py"]