FROM --platform=${BUILDPLATFORM} registry.wso2.com/wso2-apim/am-universal-gw:4.6.0.13

ARG USER=wso2carbon
ARG USER_HOME=/home/${USER}
ARG WSO2_SERVER_NAME=wso2am-universal-gw
ARG WSO2_SERVER_VERSION=4.6.0
ARG WSO2_SERVER=${WSO2_SERVER_NAME}-${WSO2_SERVER_VERSION}
ARG WSO2_SERVER_HOME=${USER_HOME}/${WSO2_SERVER}

ARG PG_VERSION=42.7.10
ARG PG_URL=https://repo1.maven.org/maven2/org/postgresql/postgresql/${PG_VERSION}/postgresql-${PG_VERSION}.jar

# Copy JDBC Postgres driver
ADD --chown=wso2carbon:wso2 ${PG_URL} ${WSO2_SERVER_HOME}/repository/components/lib