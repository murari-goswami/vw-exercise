FROM amazonlinux


RUN yum update -y \
    && yum install -y \
        zip \
        wget \
        unzip \
        tar \
        bzip2 \
        gzip \
    && yum clean all

RUN wget -q -O /tmp/AirQualityUCI.ZIP \
        --no-cookies \
        --no-check-certificate \
        "http://archive.ics.uci.edu/ml/machine-learning-databases/00360/AirQualityUCI.zip" \
    && unzip /tmp/AirQualityUCI.ZIP \
    && yum clean all


    # Derived from official mysql image (our base image)
FROM mysql

# Add a database
ENV MYSQL_DATABASE analyticsdb
ENV MYSQL_ROOT_PASSWORD admin

# Create Database
RUN	mkdir -p /usr/sql/
RUN	chmod 644 /usr/sql/

# Add the content of the sql-scripts/ directory to your image
COPY CreateTable.sql /usr/sql/CreateTable.sql
COPY InsertData.sql /usr/sql/InsertData.sql

RUN mysql start && \
    mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "CREATE DATABASE ${MYSQL_DATABASE}" && \
    mysql -u root -p${MYSQL_ROOT_PASSWORD} -D ${MYSQL_DATABASE} < /usr/sql/CreateTable.sql && \
    mysql -u root -p${MYSQL_ROOT_PASSWORD} -D ${MYSQL_DATABASE} < /usr/sql/InsertData.sql && \
    rm -rd /usr/sql


ENV PACKAGE_URL https://repo.mysql.com/yum/mysql-8.0-community/docker/x86_64/mysql-community-server-minimal-8.0.2-0.1.dmr.el7.x86_64.rpm
ENV MYSQL_ROOT_PASSWORD=root
ENV MYSQL_ROOT_USER=root
ENV DEBIAN_FRONTEND=noninteractive

RUN rpmkeys \
    && yum update -y \
    && yum install -y \
        zip \
        wget \
        unzip \
        tar \
        bzip2 \
        gzip \
        ${PACKAGE_URL} \
        libpwquality \
    && yum clean all

RUN mkdir /docker-entrypoint-initdb.d

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /entrypint.sh
ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 3306 3306

RUN wget -q -O /tmp/AirQualityUCI.ZIP \
        --no-cookies \
        --no-check-certificate \
        "http://archive.ics.uci.edu/ml/machine-learning-databases/00360/AirQualityUCI.zip" \
    && cd /tmp/ \
    && unzip /tmp/AirQualityUCI.ZIP \
    && rm -f AirQualityUCI.ZIP AirQualityUCI.xlsx \
    && yum clean all
