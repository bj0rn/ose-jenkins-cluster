FROM openshift/jenkins-1-centos7
COPY plugins.txt /opt/openshift/configuration/plugins.txt
RUN /usr/local/bin/plugins.sh /opt/openshift/configuration/plugins.txt


ADD config-cloud.xml credentials.xml /opt/openshift/
RUN touch /opt/openshift/configuration/config.xml

# Install additional plugins and modify files
RUN sed -i "s|<slaveAgentPort>.*</slaveAgentPort>|<slaveAgentPort>50000</slaveAgentPort>|" "/opt/openshift/configuration/config.xml" && \
		  sed -i "s|<numExecutors>.*</numExecutors>|<numExecutors>0</numExecutors>|" "/opt/openshift/configuration/config.xml" && \
		  sed -i "/<slaves\/>/r /opt/openshift/config-cloud.xml" "/opt/openshift/configuration/config.xml" && \
		  rm -rf /opt/openshift/config-cloud.xml /opt/openshift/credentials.xml
		  

# Add Openshift command line interface (OC)
RUN curl -L --insecure https://github.com/openshift/origin/releases/download/v1.1.1/openshift-origin-server-v1.1.1-e1d9873-linux-64bit.tar.gz | tar -zx && \
    mv openshift-origin-server-v1.1.1-e1d9873-linux-64bit/oc /usr/local/bin && \
    rm -r openshift-origin-server-v1.1.1-e1d9873-linux-64bit



