# COmanage Registry Dockerfile
#
# Portions licensed to the University Corporation for Advanced Internet
# Development, Inc. ("UCAID") under one or more contributor license agreements.
# See the NOTICE file distributed with this work for additional information
# regarding copyright ownership.
#
# UCAID licenses this file to you under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with the
# License. You may obtain a copy of the License at:
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
FROM php:7.3.13-apache-buster

# Official PHP image with Apache HTTPD includes
# --with-openssl
# --with-mbstring
# but xls, pdo, pdo_mysql, pdo_pgsql, mysqli, pgsql, 
# and ldap extensions must be built.
RUN apt-get update && apt-get install -y \
         libldap-2.4-2 \
         libldap2-dev \
         libmariadbclient-dev \
         libpq-dev \
         libxml2 \
         libxslt1-dev \
         libxslt1.1 \
         ssl-cert \
         wget \
         zlib1g \
         libpcre3-dev \
      && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
      && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu \
      && docker-php-ext-configure mysqli \
      && docker-php-ext-install xsl pdo pdo_mysql pdo_pgsql mysqli pgsql ldap \
      && docker-php-source delete \
      && apt-get purge -y \
         libldap2-dev \
         libmysqlclient-dev \
         libpq-dev \
      && apt-get clean

ARG COMANAGE_REGISTRY_VERSION
ENV COMANAGE_REGISTRY_VERSION ${COMANAGE_REGISTRY_VERSION:-develop}
LABEL comanage_registry_version=${COMANAGE_REGISTRY_VERSION}

ARG COMANAGE_REGISTRY_SRC_URL
ENV COMANAGE_REGISTRY_SRC_URL=${COMANAGE_REGISTRY_SRC_URL:-https://github.com/Internet2/comanage-registry/archive/${COMANAGE_REGISTRY_VERSION}.tar.gz}

ARG COMANAGE_REGISTRY_DIR
ENV COMANAGE_REGISTRY_DIR ${COMANAGE_REGISTRY_DIR:-/srv/comanage-registry}
LABEL comanage_registry_dir=${COMANAGE_REGISTRY_DIR}

WORKDIR $COMANAGE_REGISTRY_DIR

RUN mkdir -p ${COMANAGE_REGISTRY_DIR} \
      && wget -O comanage.tar.gz ${COMANAGE_REGISTRY_SRC_URL} \
      && tar -zxf comanage.tar.gz -C ${COMANAGE_REGISTRY_DIR} --strip-components=1 \
      && rm -f comanage.tar.gz \
      && rm -f ${COMANAGE_REGISTRY_DIR}/app/tmp \
      && cp -r ${COMANAGE_REGISTRY_DIR}/app/tmp.dist ${COMANAGE_REGISTRY_DIR}/app/tmp \
      && chown -R www-data:www-data ${COMANAGE_REGISTRY_DIR}/app/tmp \
      && cd /var/www/html \
      && ln -s ${COMANAGE_REGISTRY_DIR}/app/webroot registry \
      && rm -rf ${COMANAGE_REGISTRY_DIR}/local/*

RUN a2enmod headers \
      && a2enmod ssl \
      && a2enmod rewrite \
      && a2dissite 000-default.conf \
      && a2disconf other-vhosts-access-log \
      && cd /etc/apache2 \
      && ln -s /etc/ssl/certs/ssl-cert-snakeoil.pem cert.pem \
      && ln -s /etc/ssl/private/ssl-cert-snakeoil.key privkey.pem

COPY apache-include-directory-registry /etc/apache2/
COPY apache-include-virtual-host-port443-base /etc/apache2/
COPY apache-include-virtual-host-port80-redirect /etc/apache2/

COPY comanage_utils.sh /usr/local/lib/
COPY comanage_shibboleth_sp_utils.sh /usr/local/lib/
COPY docker-comanage-entrypoint /usr/local/bin/

VOLUME ${COMANAGE_REGISTRY_DIR}/local

EXPOSE 80 443

# Allow values for first administrator bootstrapped into the
# platform to be specified at image build time, in addition to
# being injected at run time through the entrypoint script.
ARG COMANAGE_REGISTRY_ADMIN_GIVEN_NAME
ARG COMANAGE_REGISTRY_ADMIN_FAMILY_NAME
ARG COMANAGE_REGISTRY_ADMIN_USERNAME
ARG COMANAGE_REGISTRY_ENABLE_POOLING

# Set simple defaults for first administrator bootstrapped into the
# platform to make simple evaluation of the platform easier.
ENV COMANAGE_REGISTRY_ADMIN_GIVEN_NAME ${COMANAGE_REGISTRY_ADMIN_GIVEN_NAME:-Registry}
ENV COMANAGE_REGISTRY_ADMIN_FAMILY_NAME ${COMANAGE_REGISTRY_ADMIN_FAMILY_NAME:-Admin}
ENV COMANAGE_REGISTRY_ADMIN_USERNAME ${COMANAGE_REGISTRY_ADMIN_USERNAME:-registry.admin}
ENV COMANAGE_REGISTRY_ENABLE_POOLING ${COMANAGE_REGISTRY_ENABLE_POOLING:-No}

ENTRYPOINT ["docker-comanage-entrypoint"]

CMD ["apache2-foreground"]
