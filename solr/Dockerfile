FROM tomcat:8.0-jre8
LABEL io.github.islandora-collaboration-group.name="isle-solr" \
      io.github.islandora-collaboration-group.service.name="Apache Solr" \
      io.github.islandora-collaboration-group.service.version="4.10.4" \
      io.github.islandora-collaboration-group.tomcat.version="tomcat:8.0-jre8" \
      io.github.islandora-collaboration-group.vcs-url="git@github.com:Islandora-Collaboration-Group/ISLE.git" \
      io.github.islandora-collaboration-group.vendor="Islandora Collaboration Group (ICG) - islandora-consortium-group@googlegroups.com" \
      io.github.islandora-collaboration-group.description="Apache Solr and Dependencies" \
      io.github.islandora-collaboration-group.license="Apache-2.0" \
      io.github.islandora-collaboration-group.version="Alpha-2.0.1" 

###
# Copy over tomcat files for configuration
COPY tomcat/ /usr/local/tomcat/conf/

###
# Solr Installation
#
# Set up environmental variables for tomcat & dependencies installation
# ENV  JAVA_OPTS="-Djava.awt.headless=true -Xmx1024m -XX:MaxPermSize=256m -XX:+UseConcMarkSweepGC -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Addresses=true" \
#      CATALINA_OPTS="-Djava.net.preferIPv4Stack=true" \

###
# https://github.com/phusion/baseimage-docker/issues/58
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections \
    && cd /tmp \
    && wget "http://archive.apache.org/dist/lucene/solr/4.10.4/solr-4.10.4.tgz" \
    && tar -xzf /tmp/solr-4.10.4.tgz \
    && cp -v /tmp/solr-4.10.4/dist/solr-4.10.4.war /usr/local/tomcat/webapps/solr.war \
    && /usr/bin/unzip -o /usr/local/tomcat/webapps/solr.war -d /usr/local/tomcat/webapps/solr/ \
    && cp -rv /tmp/solr-4.10.4/example/solr/. /usr/local/solr/ \
    && cp -rv /tmp/solr-4.10.4/example/lib/ext/. /usr/local/tomcat/webapps/solr/WEB-INF/lib/ \
    && cp -v /tmp/solr-4.10.4/contrib/analysis-extras/lib/icu4j-53.1.jar /usr/local/tomcat/webapps/solr/WEB-INF/lib/ \
    && cp -v /tmp/solr-4.10.4/contrib/analysis-extras/lucene-libs/lucene-analyzers-icu-4.10.4.jar /usr/local/tomcat/webapps/solr/WEB-INF/lib/ \
    && rm -rf /tmp/solr-4.10.4

COPY solr/solr.xml /usr/local/tomcat/conf/Catalina/localhost/solr.xml
COPY solr/log4j.properties /usr/local/tomcat/webapps/solr/WEB-INF/classes/log4j.properties
COPY collection1/ /usr/local/solr/collection1/conf/

EXPOSE 8080 8091
CMD ["catalina.sh", "run"]
