FROM alpine:3.6

MAINTAINER Mathew J Kelly <mjtk.ktjm@gmail.com>

RUN apk update && \
    apk add vim && \ 
    apk add gcc python3 python3-dev uwsgi-python3 

ADD requirements.txt /tmp/
ADD ./src/app.py /srv
ADD ./app_uwsgi.ini /etc/uwsgi/conf.d/
RUN ln -sf /dev/stderr /var/log/uwsgi/app_uwsgi.log

RUN /usr/bin/pip3 install --upgrade pip && \
    /usr/bin/pip3 install -r /tmp/requirements.txt

WORKDIR /srv

CMD ["uwsgi", "--ini", "/etc/uwsgi/conf.d/app_uwsgi.ini"]
