FROM guacamole/guacamole:1.0.0

LABEL maintainer="Support@quali.com" url="https://support.quali.com"


ENV GUACAMOLE_HOME /opt/guacamole/.guacamole
ENV GUACAMOLE_PROPERTIES guacamole.properties


ENV QUALIX_S3_AUTH_FILE qualix-0.9.10.tar.gz
ENV QUALIX_S3_URL https://s3.amazonaws.com/quali-prod-binaries/${QUALIX_S3_AUTH_FILE}

ENV QUALIX_GUACAMOLE_VERSION 0.9.10-incubating
ENV QUALIX_AUTH_PACK_NAME guacamole-auth-quali-${QUALIX_GUACAMOLE_VERSION}

ENV KEYSTORE_PASS 123123
ENV GUACAMOLE_WAR_NAME remote
ENV CERT_LOC /usr/share/tomcat
##########################################################################################################################################


######################## create Certificate ##############################################################################################
WORKDIR ${CERT_LOC}
RUN keytool -genkeypair -noprompt -alias Tomcat -keyalg RSA -dname "CN=quali, OU=quali, O=quali, L=quali, S=quali, C=IL" -keystore .keystore -storepass ${KEYSTORE_PASS} -keypass ${KEYSTORE_PASS} -validity 365000
WORKDIR /usr/local/tomcat/conf

RUN sed -i '/<Service name="Catalina">/a <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol" maxThreads="150" SSLEnabled="true" scheme="https" secure="true" keystoreFile="'${CERT_LOC}'/.keystore" keystorePass="'${KEYSTORE_PASS}'" clientAuth="false" sslProtocol="TLS" />' server.xml
##########################################################################################################################################


######################## Get quali qualix package from S3 #################################################################################
RUN mkdir -p ${GUACAMOLE_HOME}/extensions 
# RUN wget -qO- ${QUALIX_S3_URL} | tar -xzf - 
COPY qualix-1.0.0.tar.gz /tmp

RUN tar -xzf /tmp/qualix-1.0.0.tar.gz
RUN cp guacamole-auth-quali-1.0.0/guacamole-auth-quali-1.0.0.jar ${GUACAMOLE_HOME}/extensions/
RUN rm -rf  guacamole-auth-quali-1.0.0
# RUN cp ${QUALIX_AUTH_PACK_NAME}/${QUALIX_AUTH_PACK_NAME}.jar ${GUACAMOLE_HOME}/extensions/
# RUN rm -r ${QUALIX_AUTH_PACK_NAME}

ENV redirect '<% response.sendRedirect("/remote"); %>'
RUN sed -i '1s|^|'"$redirect"'\n|' /usr/local/tomcat/webapps/ROOT/index.jsp
##########################################################################################################################################



######################## Copy configuration ##############################################################################################
RUN echo "auth-provider: net.sourceforge.guacamole.net.auth.quali.QualiProvider"  > ${GUACAMOLE_HOME}/${GUACAMOLE_PROPERTIES}
RUN mkdir -p ${GUACAMOLE_HOME}/files/
RUN chmod -R 777 ${GUACAMOLE_HOME}/
RUN chmod -R 777 ${GUACAMOLE_HOME}/files/
##########################################################################################################################################

RUN  mv /opt/guacamole/guacamole.war /opt/guacamole/${GUACAMOLE_WAR_NAME}.war
RUN ln -s /opt/guacamole/${GUACAMOLE_WAR_NAME}.war /usr/local/tomcat/webapps/${GUACAMOLE_WAR_NAME}.war



RUN apt-get update
RUN apt-get install -y  sysv-rc-conf
RUN sysv-rc-conf --level 234 tomcat on
RUN apt-get install -y vim
