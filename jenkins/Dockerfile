FROM openshift/jenkins-16-centos7

ENV HOME=/var/lib/jenkins 


ADD config-cloud.xml credentials.xml /opt/openshift/

# Install additional plugins and modify files
RUN mkdir -p /opt/openshift/plugins && \
          cat /opt/openshift/credentials.xml > /opt/openshift/configuration/credentials.xml && \
          curl -sSL -f https://updates.jenkins-ci.org/download/plugins/swarm/2.0/swarm.hpi -o /opt/openshift/plugins/swarm.jpi && \
		  curl -sSL -f https://updates.jenkins-ci.org/download/plugins/durable-task/1.7/durable-task.hpi -o /opt/openshift/plugins/durable-task.jpi && \
		  curl -sSL -f https://updates.jenkins-ci.org/download/plugins/credentials/1.24/credentials.hpi -o /opt/openshift/plugins/credentials.jpi && \
		  curl -sSL -f https://updates.jenkins-ci.org/download/plugins/kubernetes/0.5/kubernetes.hpi -o /opt/openshift/plugins/kubernetes.jpi && \
		  touch /opt/openshift/plugins/credentials.jpi.pinned && \
		  sed -i "s|<slaveAgentPort>.*</slaveAgentPort>|<slaveAgentPort>50000</slaveAgentPort>|" "/opt/openshift/configuration/config.xml" && \
		  sed -i "s|<numExecutors>.*</numExecutors>|<numExecutors>0</numExecutors>|" "/opt/openshift/configuration/config.xml" && \
		  sed -i "/<slaves\/>/r /opt/openshift/config-cloud.xml" "/opt/openshift/configuration/config.xml" && \
		  chmod -R og+rw /opt/openshift/plugins && \
		  rm -rf /opt/openshift/config-cloud.xml /opt/openshift/credentials.xml

USER root

RUN chmod -R 777 ${HOME} && chown -R 1001:0 ${HOME}

# We base this Dockerfile on the openshift/jenkins with slight modifications and additions. We need to
# install the tools that the openshift/jenkins does and has embedded. These files are copied from
# https://github.com/openshift/jenkins
COPY openshift-jenkins/openshift /opt/openshift
COPY openshift-jenkins/jenkins/* /usr/local/bin/
COPY openshift-jenkins/s2i /usr/libexec/s2i
RUN sed -i -- 's|cp -r /opt/openshift/configuration/\* \${JENKINS_HOME}|cp -rT /opt/openshift/configuration ${JENKINS_HOME}|g' /usr/libexec/s2i/run

RUN chown -R 1001:0 /opt/openshift && \
    /usr/local/bin/fix-permissions /opt/openshift && \
    /usr/local/bin/fix-permissions /var/lib/jenkins

# Add our own set of plugins
COPY config /opt/openshift/configuration
RUN /usr/local/bin/plugins.sh /opt/openshift/configuration/plugins.txt
# We need to pin the version of the credentials plugin that we download from plugins.txt, or else it will be
# overwritten by the credentials plugin version that is embedded in jenkins (which, at the time of this writing is
# older than the version we want to use).
RUN touch /opt/openshift/plugins/credentials.jpi.pinned


USER 1001
