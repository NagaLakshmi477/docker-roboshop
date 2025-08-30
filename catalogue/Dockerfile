FROM node:20
WORKDIR /opt/server
COPY  package.json .
COPY *.js .
# now we can install dependies
RUN npm install
# for sysytemctl files we need to setup env variables
# dns name comes from docker inspect
ENV MONGO="true"\
MONGO_URL="mongodb://mongodb:27017/catalogue"
# here we will use CMD beacuse we need to login and check 
# we alraedy in /appfloder 
CMD [ "node" ," server.js"]