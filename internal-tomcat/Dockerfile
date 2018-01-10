FROM tomcat:9.0.2-alpine

COPY ./tomcat-users.xml /usr/local/tomcat/conf/tomcat-users.xml
COPY ./manager-context.xml /usr/local/tomcat/webapps/manager/META-INF/context.xml

EXPOSE 8080

CMD ["catalina.sh", "run"]
