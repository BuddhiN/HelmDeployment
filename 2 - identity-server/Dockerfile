FROM --platform=${BUILDPLATFORM} registry.wso2.com/wso2-is/is:7.2.0.11

ARG USER=wso2carbon
ARG USER_HOME=/home/${USER}
ARG WSO2_SERVER_NAME=wso2is
ARG WSO2_SERVER_VERSION=7.2.0
ARG WSO2_SERVER=${WSO2_SERVER_NAME}-${WSO2_SERVER_VERSION}
ARG WSO2_SERVER_HOME=${USER_HOME}/${WSO2_SERVER}

ARG PG_VERSION=42.7.10
ARG PG_URL=https://repo1.maven.org/maven2/org/postgresql/postgresql/${PG_VERSION}/postgresql-${PG_VERSION}.jar

# Add notification event handler JAR for API Manager integration
ADD --chown=wso2carbon:wso2 https://maven.wso2.org/nexus/content/repositories/releases/org/wso2/km/ext/wso2is/wso2is.notification.event.handlers/2.0.5/wso2is.notification.event.handlers-2.0.5.jar ${WSO2_SERVER_HOME}/repository/components/dropins

# Copy JDBC Postgres driver
ADD --chown=wso2carbon:wso2 ${PG_URL} ${WSO2_SERVER_HOME}/repository/components/lib