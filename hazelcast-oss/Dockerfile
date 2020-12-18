FROM maven:3.6-openjdk-11

WORKDIR /tmp/build

# Versions of Hazelcast Jet and Hazelcast IMDG plugins
ARG HZ_VERSION=4.1
ARG CACHE_API_VERSION=1.1.1
ARG JMX_PROMETHEUS_AGENT_VERSION=0.14.0
ARG LOG4J2_VERSION=2.13.3
ARG HZ_EUREKA_VERSION=2.0.1

ADD dependency-copy.xml /tmp/build/

# Download and install Hazelcast Jet and Hazelcast plugins
#(hazelcast-eureka, jcache-api) with dependencies
RUN mvn -f dependency-copy.xml \
      -Dhazelcast-eureka-version=${HZ_EUREKA_VERSION} \
      -Djmx-prometheus-agent-version=${JMX_PROMETHEUS_AGENT_VERSION} \
      -Dlog4j2-version=${LOG4J2_VERSION} \
      -Dhz-version=${HZ_VERSION} \
      -Djcache-version=${CACHE_API_VERSION} \
      -Dmaven.wagon.http.retryHandler.count=3 \
      dependency:copy-dependencies && \
      mv hazelcast-oss-${HZ_VERSION} hazelcast-oss 


FROM alpine:3.12.1

# Build constants
ARG HZ_HOME="/opt/hazelcast"

COPY --from=0 /tmp/build/hazelcast-oss ${HZ_HOME}/

# Runtime variables
ENV HZ_HOME="${HZ_HOME}" \
    CLASSPATH_DEFAULT="${HZ_HOME}/*:${HZ_HOME}/lib/*" \
    JAVA_OPTS_DEFAULT="-Djava.net.preferIPv4Stack=true -Dhazelcast.logging.type=log4j2 -Dlog4j.configurationFile=${HZ_HOME}/log4j2.properties -XX:MaxRAMPercentage=80.0 -XX:MaxGCPauseMillis=5 --add-modules java.se --add-exports java.base/jdk.internal.ref=ALL-UNNAMED --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.management/sun.management=ALL-UNNAMED --add-opens jdk.management/com.sun.management.internal=ALL-UNNAMED" \
    PROMETHEUS_PORT="" \
    PROMETHEUS_CONFIG="${HZ_HOME}/jmx_agent_config.yaml" \
    LOGGING_LEVEL="" \
    CLASSPATH="" \
    JAVA_OPTS=""

# Expose port
EXPOSE 5701

COPY *.sh *.yaml *.jar *.properties ${HZ_HOME}/

# Install
RUN echo "Installing new APK packages" \
    && apk add --no-cache openjdk11-jre  \
    && cd "${HZ_HOME}/lib" \
    && echo "Granting read permission to ${HZ_HOME}" \
    && chmod -R +r ${HZ_HOME} \
    && echo "Setting Pardot ID to 'docker'" \
    && echo 'hazelcastDownloadId=docker' > "${HZ_HOME}/hazelcast-download.properties" \
    && echo "Cleaning APK packages" \
    && rm -rf /var/cache/apk/*

WORKDIR ${HZ_HOME}

RUN addgroup -S hazelcast && adduser -S hazelcast -G hazelcast
USER root

# Start Hazelcast server
CMD ["/opt/hazelcast/start-hazelcast.sh"]
