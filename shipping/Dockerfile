FROM maven
WORKDIR /opt/server
COPY pom.xml .
COPY src /opt/server/src

RUN mvn clean package
# it will create target floder in this we have jsr files(default)
#  we will take the file outside
RUN mv /opt/server/target/shipping-1.0.jar /opt/server/shipping.jar
ENV CART_ENDPOINT="cart:8080"\
    DB_HOST="mysql"
CMD ["java", "-jar", "shipping.jar"]

