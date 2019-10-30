FROM alpine
RUN apk update \
  && apk add openjdk11-jdk \
  && rm -rf /var/cache/apk/*
ENTRYPOINT ["tail", "-f", "/dev/null"]