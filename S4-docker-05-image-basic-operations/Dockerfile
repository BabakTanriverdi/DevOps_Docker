FROM ubuntu

WORKDIR /app
RUN apt-get update -y
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN pip3 install flask --break-system-packages
COPY welcome.py .
EXPOSE 80
CMD ["python3", "welcome.py"]