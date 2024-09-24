FROM maven:3.8.5-openjdk as javabuilder

WORKDIR /opt/jmx_exporter
COPY . .
RUN mvn package -DskipTests

FROM anapsix/alpine-java:9 as scratch

RUN apk update && apk upgrade && apk --update add curl && rm -rf /tmp/* /var/cache/apk/*

ENV VERSION 1.0.0
ENV JAR jmx_prometheus_httpserver-$VERSION.jar
COPY --from=javabuilder /opt/jmx_exporter/jmx_prometheus_httpserver/target/jmx_prometheus_httpserver-$VERSION-SNAPSHOT.jar /opt/jmx_exporter/$JAR
COPY build/start.sh /opt/jmx_exporter
COPY build/config.yml /opt/jmx_exporter
RUN chmod +x /opt/jmx_exporter/start.sh

CMD ["/opt/jmx_exporter/start.sh"]

FROM quay.io/sysdig/sysdig-mini-ubi9:1.3.11 as ubi

RUN microdnf --assumeyes --disableplugin=subscription-manager --nodocs update && \
    microdnf --assumeyes --disableplugin=subscription-manager --nodocs install java-1.8.0-openjdk-headless && \
    microdnf --assumeyes --disableplugin=subscription-manager --nodocs clean all && \
    rm -rf /usr/lib/python3.6 && \
    rm -rf /var/cache/yum

ENV VERSION 1.0.0
ENV JAR jmx_prometheus_httpserver-$VERSION.jar

COPY --from=javabuilder /opt/jmx_exporter/jmx_prometheus_httpserver/target/jmx_prometheus_httpserver-$VERSION-SNAPSHOT.jar /opt/jmx_exporter/$JAR
COPY --chown=sysdig:sysdig build/start.sh /opt/jmx_exporter
COPY --chown=sysdig:sysdig build/config.yml /opt/jmx_exporter

RUN chmod +x /opt/jmx_exporter/start.sh

USER sysdig:sysdig

CMD ["/opt/jmx_exporter/start.sh"]
