FROM node:20
WORKDIR /opt/server
#copy the json and js file to the current dir
COPY package.json .
COPY *.js .
RUN npm install
ENV REDIS_URL="redis" \
    CATALOGUE_HOST="catalogue" \
    CATALOGUE_PORT="8080"
CMD [ "node","server.js" ]