FROM mhart/alpine-node:7.4.0

WORKDIR /srv
ADD ./src .
RUN npm install

EXPOSE 3000
CMD ["node", "server.js"]
