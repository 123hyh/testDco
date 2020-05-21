FROM java:8

RUN mkdir -p /home/www/express

WORKDIR /home/www/express

COPY . /home/www/express

EXPOSE 8196
ENTRYPOINT [ "nohup java -jar" ]
CMD [ "user-0.0.1-SNAPSHOT.jar" ]