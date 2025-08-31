FROM maven
WORKDIR /opt/server
COPY pom.xml .
COPY src .
RUN mvn clean package
# it will create target floder in this we have jsr files(default)
#  we will take the file outside
RUN mv /opt/server/target/shipping-1.0.jar /opt/server/shipping.jar