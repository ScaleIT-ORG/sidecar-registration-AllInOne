FROM node:boron

WORKDIR /opt/app

RUN apt-get update
ADD package.json /tmp/package.json
RUN cd /tmp && npm install
RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/
ADD ./ /opt/app/

EXPOSE 3000

CMD ["./sidecar.sh"]
