FROM openjdk:7-jdk
MAINTAINER UCF Project <ucf-project@alterway.fr>

# ##########################################################
#
#                      DISCLAIMER
#
# !!!!DO NOT USE THIS IN PRODUCTION!!!!!
# For testing purposes only.
#
# For Asqatasun installation visit:
# http://doc.asqatasun.org/en/10_Install_doc/
#
# ##########################################################

USER root

ENV DATABASE_DBNAME="asqatasun" \
    DATABASE_HOST="localhost" \
    DATABASE_USER="asqatasun" \
    DATABASE_PASSWD="asqaP4sswd" \
    MYSQL_ROOT_PASSWD="mysqlRootPassword" \
    ASQA_RELEASE="4.0.3" \
    DEBIAN_FRONTEND="noninteractive" \
    FIREFOX_BASENAME="firefox-31.4.0esr"

WORKDIR /root

# Install pre requisites

## JDK and JRE => base docker image

## MySQL

RUN apt update && apt install -y --no-install-recommends debconf && \
    echo "mysql-server mysql-server/root_password password ${MYSQL_ROOT_PASSWD}" | debconf-set-selections && \
    echo "mysql-server mysql-server/root_password_again password ${MYSQL_ROOT_PASSWD}" | debconf-set-selections && \
    apt install -y --no-install-recommends mysql-server

ADD cnf/asqatasun.cnf /etc/mysql/conf.d/asqatasun.cnf

RUN service mysql restart && \
    mysql -u root --password="${MYSQL_ROOT_PASSWD}" --execute="GRANT USAGE ON * . * TO '${DATABASE_USER}'@'${DATABASE_HOST}' IDENTIFIED BY '${DATABASE_PASSWD}'; \
        CREATE DATABASE IF NOT EXISTS ${DATABASE_DBNAME} DEFAULT CHARACTER SET utf8; \
        GRANT ALL PRIVILEGES ON ${DATABASE_DBNAME} . * TO '${DATABASE_USER}'@'${DATABASE_HOST}'; \
        FLUSH PRIVILEGES;"

## XVFB

RUN apt install -y --no-install-recommends xvfb
ADD cnf/xvfb /etc/init.d/xvfb
RUN chmod a+x /etc/init.d/xvfb && update-rc.d xvfb defaults

## Firefox

RUN apt install -y --no-install-recommends libdbus-glib-1-2 \
                                           wget && \
    cd /opt/ && \
    wget --quiet "http://download.cdn.mozilla.net/pub/mozilla.org/firefox/releases/31.4.0esr/linux-$(uname -m)/en-US/${FIREFOX_BASENAME}.tar.bz2" && \
    tar xvfj ${FIREFOX_BASENAME}.tar.bz2 && \
    rm ${FIREFOX_BASENAME}.tar.bz2

# Install Asqatasun

RUN cd /root/ && \
    wget --quiet https://github.com/Asqatasun/Asqatasun/releases/download/v${ASQA_RELEASE}/asqatasun-runner-${ASQA_RELEASE}.i386.tar.gz && \
    tar xvfz asqatasun-runner-${ASQA_RELEASE}.i386.tar.gz && \
    rm asqatasun-runner-${ASQA_RELEASE}.i386.tar.gz && \
    mv asqatasun*/ ./asqatasun/ && \
    sed -e '13,15d' -i asqatasun/conf/context/asqatasun.conf && \
    sed "12 a jdbc.url=jdbc:mysql://localhost:3306/${DATABASE_DBNAME}" -i asqatasun/conf/context/asqatasun.conf && \
    sed "13 a jdbc.username=${DATABASE_USER}" -i asqatasun/conf/context/asqatasun.conf && \
    sed "14 a jdbc.password=${DATABASE_PASSWD}" -i asqatasun/conf/context/asqatasun.conf

# Populate database

RUN service mysql start && \
    mysql -u ${DATABASE_USER} -p${DATABASE_PASSWD} ${DATABASE_DBNAME} < asqatasun/install/engine/sql/asqatasun-20-create-tables.sql && \
    mysql -u ${DATABASE_USER} -p${DATABASE_PASSWD} ${DATABASE_DBNAME} < asqatasun/install/engine/sql/asqatasun-30-insert.sql && \
    mysql -u ${DATABASE_USER} -p${DATABASE_PASSWD} ${DATABASE_DBNAME} < asqatasun/install/rules/sql/10-rules-resources-insert.sql && \
    # Then, modify the 3 following scripts, install/rules/sql/accessiweb2.2-insert.sql, install/rules/sql/rgaa2.2-insert.sql and install/rules/sql/rgaa3.0-insert.sql by removing the lines starting with "INSERT IGNORE INTO TGSI_REFERENTIAL" and execute the scripts
    sed -e '10,11d' -i asqatasun/install/rules/sql/accessiweb2.2-insert.sql && \
    sed -e '6,7d' -i asqatasun/install/rules/sql/seo1.0-insert.sql && \
    sed -e '4,5d' -i asqatasun/install/rules/sql/rgaa3.0-insert.sql && \
    mysql -u ${DATABASE_USER} -p${DATABASE_PASSWD} ${DATABASE_DBNAME} < asqatasun/install/rules/sql/accessiweb2.2-insert.sql && \
    # mysql -u ${DATABASE_USER} -p${DATABASE_PASSWD} ${DATABASE_DBNAME} < asqatasun/install/rules/sql/seo1.0-insert.sql && \
    mysql -u ${DATABASE_USER} -p${DATABASE_PASSWD} ${DATABASE_DBNAME} < asqatasun/install/rules/sql/rgaa3.0-insert.sql

WORKDIR /root/asqatasun/

# Start all services

CMD service mysql start && \
    sleep 5 && \
    service xvfb start && \
    tail -f /root/asqatasun/logs/*

# Example of usage
# docker run --name asqatasun-cli -d docker-asqatasun-cli
# docker exec asqatasun-cli ./bin/asqatasun.sh -f /opt/firefox/firefox-bin -d 99 -r Rgaa30 http://alterway.fr/
# docker exec asqatasun-cli ./bin/asqatasun.sh -f /opt/firefox/firefox-bin -d 99 -r Rgaa30 -t File a.html
