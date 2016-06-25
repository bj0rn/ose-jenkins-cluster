FROM openshift/jenkins-1-centos7
USER root
COPY plugins.txt /opt/openshift/configuration/plugins.txt
COPY config.xml /opt/openshift/configuration/config.xml

RUN /usr/local/bin/plugins.sh /opt/openshift/configuration/plugins.txt

RUN yum install -y git tar zip unzip which java-1.8.0-openjdk-devel

RUN yum -y --disableplugin=fastestmirror groupinstall "Development Tools" && yum clean all

ADD config-cloud.xml credentials.xml /opt/openshift/

# Add Openshift command line interface (OC)
RUN curl -L --insecure https://github.com/openshift/origin/releases/download/v1.1.1/openshift-origin-server-v1.1.1-e1d9873-linux-64bit.tar.gz | tar -zx && \
    mv openshift-origin-server-v1.1.1-e1d9873-linux-64bit/oc /usr/local/bin && \
    rm -r openshift-origin-server-v1.1.1-e1d9873-linux-64bit





