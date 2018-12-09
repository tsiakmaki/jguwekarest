FROM tomcat:8.0-jre8

# install some basic packages
RUN apt-get -qq -y update && \
	 apt-get -qq -y install wget net-tools vim curl git default-jdk maven

#
# deploy jguwekarest 
#

# remove preinstalled webapps, if rm works on your docker
#RUN rm -fR /usr/local/tomcat/webapps/ROOT
#RUN rm -fR /usr/local/tomcat/webapps/host-manager
#RUN rm -fR /usr/local/tomcat/webapps/manager
#RUN rm -fR /usr/local/tomcat/webapps/docs
#RUN rm -fR /usr/local/tomcat/webapps/example

# build weka war, copy the application war file
RUN cd /tmp && git clone https://github.com/jguwekarest/jguwekarest.git && \
        cd jguwekarest && mvn package && cp target/weka_rs-0.3.0.war /usr/local/tomcat/webapps/weka_rc.war
# copy and unzip the application war file
RUN unzip -d /usr/local/tomcat/webapps/weka_rc /usr/local/tomcat/webapps/weka_rc.war && rm -f /usr/local/tomcat/webapps/weka_rc.war

# mongodb settings
RUN sed -i "s/db.host=mongodb/db.host=myhostname/" /usr/local/tomcat/webapps/weka_rc/WEB-INF/classes/config/db.properties

# 
# ssl and tomcat using pre-existing certificate and private key from a ca
# 
# copy root, intermediates, server's certificate, and the private key file 
# intermediate authority
COPY intermediate.pem /usr/local/tomcat/conf/intermediate.pem
# root authority 
COPY root.pem /usr/local/tomcat/conf/root.pem
# server's certificate in pem format
COPY cert.pem /usr/local/tomcat/conf/cert.pem
# private key
COPY key.pem  /usr/local/tomcat/conf/key.pem
# certs.pem contain a list of certificate authorities, starting from the intermediate authorities to the root authorities.
# —–BEGIN CERTIFICATE—–
#  the intermediate certificate here (base64)
#—–END CERTIFICATE—–
#—–BEGIN CERTIFICATE—–
# the root certificate here (base64)
#—–END CERTIFICATE—–
# cat root.pem intermediate.pem > certs.pem 
COPY certs.pem /usr/local/tomcat/conf/certs.pem

# add root and intermediates certificates to the new keystore, by default the created keystore file will be the /root/.keystore
RUN keytool -import -alias root -file /usr/local/tomcat/conf/root.pem -keypass changeit -storepass changeit -noprompt
RUN keytool -import -alias intermediate -file /usr/local/tomcat/conf/intermediate.pem -keypass changeit -storepass changeit -noprompt

# keytool does not support importing a pre-existing private key.
# convert the x.509 certificate and private key to a pkcs12
RUN openssl pkcs12 -export -in /usr/local/tomcat/conf/cert.pem -inkey /usr/local/tomcat/conf/key.pem -out /usr/local/tomcat/conf/server.p12 -name tomcat -CAfile /usr/local/tomcat/conf/certs.pem -caname root -passout pass:changeit

# import the new pkcs12 file into the java keystore
RUN keytool -importkeystore -deststorepass changeit -destkeypass changeit -destkeystore /root/.keystore -srckeystore /usr/local/tomcat/conf/server.p12 -srcstoretype PKCS12 -srcstorepass changeit -alias tomcat

# convert keystore to pkcs12 format
RUN keytool -importkeystore -srckeystore /root/.keystore -destkeystore /root/.keystore -deststoretype pkcs12 -deststorepass changeit -srcstorepass changeit

# Modify server.xml to activate the TLS service
RUN sed -i "s/<Service name=\"Catalina\">/<Service name=\"Catalina\"><Connector port=\"8443\" keystoreType=\"PKCS12\" clientAuth=\"false\" allowUnsafeLegacyRenegotiation=\"true\"  protocol=\"org.apache.coyote.http11.Http11NioProtocol\" SSLEnabled=\"true\" scheme=\"https\" secure=\"true\"  sslProtocol=\"TLS\" keystoreFile=\"\/root\/.keystore\" keystorePass=\"changeit\" sslEnabledProtocols=\"TLSv1,TLSv1.1,TLSv1.2\" ciphers=\"TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_RC4_128_SHA, TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,SSL_RSA_WITH_RC4_128_SHA\"  \/>/" /usr/local/tomcat/conf/server.xml


EXPOSE 8080
EXPOSE 8443 
CMD ["/usr/local/tomcat/bin/catalina.sh", "run"]
