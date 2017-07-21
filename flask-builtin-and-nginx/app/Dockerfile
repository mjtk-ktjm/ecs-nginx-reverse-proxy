FROM alpine:3.6

MAINTAINER Mathew J Kelly <mjtk.ktjm@gmail.com>

RUN apk update && \
    apk add vim && \ 
    apk add gcc python3 python3-dev uwsgi-python3

ADD requirements.txt /tmp/

RUN /usr/bin/pip3 install --upgrade pip && \
    /usr/bin/pip3 install -r /tmp/requirements.txt

WORKDIR /srv
ADD ./src .

STOPSIGNAL SIGQUIT

CMD ["python3", "app.py"]
