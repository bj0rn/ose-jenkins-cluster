FROM centos:centos7

ENV HOME=/var/lib/jenkins \
    JENKINS_HOME=/var/lib/jenkins

# 8080 for main web interface, 50000 for slave agents
EXPOSE 8080 50000

RUN curl http://pkg.jenkins-ci.org/redhat/jenkins.repo -o /etc/yum.repos.d/jenkins.repo && \
    rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key && \
    yum -y --setopt=tsflags=nodocs --disableplugin=fastestmirror install epel-release && \
    sed -i -- 's/\#baseurl/baseurl/g' /etc/yum.repos.d/epel.repo && \
    sed -i -- 's/mirrorlist/\#mirrorlist/g' /etc/yum.repos.d/epel.repo && \
    yum -y --setopt=tsflags=nodocs --disableplugin=fastestmirror install gettext git tar zip unzip nss_wrapper && \
    yum clean all  && \
    localedef -f UTF-8 -i en_US en_US.UTF-8

# We need to do insecure https downloads from curl because of the way certificates are handled in the SITS networks.
# We enable it globally here because the /usr/local/bin/plugins.sh that is invoked later (downloaded from
# https://codeload.github.com/openshift/jenkins/zip/master also uses curl to download plugins).
RUN mkdir -p $HOME && echo insecure > $HOME/.curlrc

# Install Java
ENV JDK_VERSION=8u66 \
    JDK_BUILD_VERSION=b17
RUN curl -LO "http://download.oracle.com/otn-pub/java/jdk/$JDK_VERSION-$JDK_BUILD_VERSION/jdk-$JDK_VERSION-linux-x64.rpm" -H 'Cookie: oraclelicense=accept-securebackup-cookie' && \
    rpm -i jdk-$JDK_VERSION-linux-x64.rpm && \
    rm -f jdk-$JDK_VERSION-linux-x64.rpm

# Add Openshift command line interface (OC)
RUN curl -L --insecure https://github.com/openshift/origin/releases/download/v1.1.3/openshift-origin-client-tools-v1.1.3-cffae05-linux-64bit.tar.gz | tar -zx && \
    mv openshift-origin-client-tools*/oc /usr/local/bin && \
    rm -rf openshift-origin-client-tools*

# Install Jenkins
RUN yum -y --setopt=tsflags=nodocs --disableplugin=fastestmirror install jenkins-1.652

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

VOLUME ["/var/lib/jenkins"]

CMD ["/usr/libexec/s2i/run"]
