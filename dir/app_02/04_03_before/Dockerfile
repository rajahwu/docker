# This is a testing image
FROM ubuntu
COPY . /app
ENV curl_bin="curl"
RUN apt -y update && apt -y install "$curl_bin"
ENTRYPOINT [ "/app/app.sh" ]
CMD [ "--agrument" ]
