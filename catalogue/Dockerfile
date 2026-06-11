FROM node:20
WORKDIR /opt/server
#copy the json and js file to the current dir
COPY package.json .
COPY *.js .
RUN npm install
ENV MONGO=true \
    MONGO_URL="mongodb://mongodb.nagalakshmi.site:27017/catalogue" 
CMD [ "node","server.js" ]