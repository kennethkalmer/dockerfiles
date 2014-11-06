FROM ubuntu:14.04

MAINTAINER Kenneth Kalmer "https://github.com/kennethkalmer/dockerfiles"

RUN echo "debconf debconf/frontend select Teletype" | debconf-set-selections &&\
    apt-get -y update &&\
    apt-get -y install software-properties-common &&\
    apt-get -y update &&\
    apt-mark hold initscripts &&\
    apt-get -y upgrade &&\
    apt-get -y install build-essential git curl wget zip \
                       libxslt-dev libcurl4-openssl-dev \
                       libssl-dev libyaml-dev libtool \
                       libxml2-dev gawk \
                       imagemagick \
                       postgresql-client \
                       libpq-dev libreadline-dev \
                       mysql-client libmysqlclient-dev \
                       openjdk-7-jre-headless &&\
    dpkg-divert --local --rename --add /sbin/initctl &&\
    sh -c "test -f /sbin/initctl || ln -s /bin/true /sbin/initctl"

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

# Jenkins installation
ENV JENKINS_VERSION 1.580.1
# define url prefix for running jenkins behind Apache (https://wiki.jenkins-ci.org/display/JENKINS/Running+Jenkins+behind+Apache)
ENV JENKINS_PREFIX /
RUN mkdir /usr/share/jenkins/
RUN useradd -d /home/jenkins -m -s /bin/bash jenkins

COPY init.groovy /tmp/WEB-INF/init.groovy.d/tcp-slave-angent-port.groovy
RUN curl -L http://mirrors.jenkins-ci.org/war-stable/$JENKINS_VERSION/jenkins.war -o /usr/share/jenkins/jenkins.war \
  && cd /tmp && zip -g /usr/share/jenkins/jenkins.war WEB-INF/init.groovy.d/tcp-slave-angent-port.groovy && rm -rf /tmp/WEB-INF

ENV JENKINS_HOME /var/jenkins_home
RUN usermod -m -d "$JENKINS_HOME" jenkins && chown -R jenkins "$JENKINS_HOME"

# Node installation
RUN sh -c "curl -sL https://deb.nodesource.com/setup | bash -" &&\
    apt-get install -y nodejs

# PhantomJS installation
RUN wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-1.9.7-linux-x86_64.tar.bz2 &&\
    tar -xjf phantomjs-1.9.7-linux-x86_64.tar.bz2 &&\
    rm phantomjs-1.9.7-linux-x86_64.tar.bz2 &&\
    cp phantomjs-1.9.7-linux-x86_64/bin/phantomjs /bin/phantomjs &&\
    rm -fr phantomjs-1.9.7-linux-x86_64

# Place for rbenv
ENV RBENV_ROOT /usr/local/rbenv

# Get rbenv installed
RUN git clone https://github.com/sstephenson/rbenv.git $RBENV_ROOT &&\
    git clone https://github.com/sstephenson/ruby-build.git $RBENV_ROOT/plugins/ruby-build &&\
    echo 'eval "$(rbenv init -)"' >> /etc/profile.d/rbenv.sh &&\
    echo 'gem: --no-document' >> /etc/gemrc

# Setup our path
ENV PATH $RBENV_ROOT/bin:$RBENV_ROOT/shims:$PATH

# Install our rubies
ENV CONFIGURE_OPTS --disable-install-doc
ADD ./versions.txt /root/versions.txt
RUN xargs -L 1 rbenv install < /root/versions.txt

# Install Bundler for each version of ruby
RUN bash -l -c 'for v in $(cat /root/versions.txt); do rbenv shell $v; gem update --system; gem install bundler; done'

# Get more application specific here
RUN apt-get install -y pdftk

# Convert jenkins home into a persistent volume
VOLUME /var/jenkins_home

# for main web interface:
EXPOSE 8080

# will be used by attached slave agents:
EXPOSE 50000

USER jenkins

COPY jenkins.sh /usr/local/bin/jenkins.sh
ENTRYPOINT ["/usr/local/bin/jenkins.sh"]
