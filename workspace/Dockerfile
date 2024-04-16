#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/Laradock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG LARADOCK_PHP_VERSION
ARG BASE_IMAGE_TAG_PREFIX=latest
FROM laradock/workspace:${BASE_IMAGE_TAG_PREFIX}-${LARADOCK_PHP_VERSION}

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

ARG LARADOCK_PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

###########################################################################
# If you're in China, or you need to change sources, will be set CHANGE_SOURCE to true in .env.
###########################################################################

ARG CHANGE_SOURCE=false
ARG UBUNTU_SOURCE=aliyun
COPY ./sources.sh /tmp/sources.sh
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    # Change application source from deb.debian.org to aliyun source
    sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list; \
    sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list; \
    sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list; \
    chmod +x /tmp/sources.sh; \
    /bin/sh -c /tmp/sources.sh; \
    fi; \
    rm -rf /tmp/sources.sh

###########################################################################
# Laradock non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} laradock && \
    useradd -l -u ${PUID} -g laradock -m laradock -G docker_env && \
    usermod -p "*" laradock -s /bin/bash && \
    apt-get install -yqq \
    libargon2-1 libidn2-0 libpcre2-8-0 libpcre3 libxml2 libzstd1 apt-utils \
    #
    #--------------------------------------------------------------------------
    # Mandatory Software's Installation
    #--------------------------------------------------------------------------
    #
    # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
    # installed on the base image 'laradock/workspace' image. If you want
    # to add more Software's or remove existing one, you need to edit the
    # base image (https://github.com/Laradock/workspace).
    #
    # next lines are here because there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
    libzip-dev zip unzip \
    # Install the zip extension
    php${LARADOCK_PHP_VERSION}-zip \
    # nasm
    nasm && \
    php -m | grep -q 'zip'

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=PRC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/laradock/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/laradock/aliases.sh && \
    chown laradock:laradock /home/laradock/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

USER laradock

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/laradock/.composer/composer.json

# Add the auth.json for magento 2 credentials
COPY ./auth.json /home/laradock/.composer/auth.json

# Make sure that ~/.composer belongs to laradock
RUN chown -R laradock:laradock /home/laradock/.composer

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Update composer
ARG COMPOSER_VERSION=2
ENV COMPOSER_VERSION ${COMPOSER_VERSION}
RUN set -eux; \
    if [ "$COMPOSER_VERSION" = "1" ] || [ "$COMPOSER_VERSION" = "2" ]; then \
    composer self-update --${COMPOSER_VERSION}; \
    else \
    composer self-update ${COMPOSER_VERSION}; \
    fi

USER laradock

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
    ;fi

# Check if auth file is disabled
ARG COMPOSER_AUTH=false
ENV COMPOSER_AUTH ${COMPOSER_AUTH}

RUN if [ ${COMPOSER_AUTH} = false ]; then \
    # remove the file
    rm /home/laradock/.composer/auth.json \
    ;fi

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
    ;fi

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Non-root user : PHPUnit path
###########################################################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER laradock

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Crontab
###########################################################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d


###########################################################################
USER root

ARG INSTALL_BZ2=false
ARG INSTALL_GMP=false
ARG INSTALL_GNUPG=false
ARG INSTALL_SSH2=false
ARG INSTALL_SOAP=false
ARG INSTALL_XSL=false
ARG PHP_VERSION=${LARADOCK_PHP_VERSION}

RUN set -eux; \
    ###########################################################################
    # BZ2:
    ###########################################################################
    if [ ${INSTALL_BZ2} = true ]; then \
    apt-get -yqq install php${LARADOCK_PHP_VERSION}-bz2; \
    fi; \
    ###########################################################################
    # GMP (GNU Multiple Precision):
    ###########################################################################
    if [ ${INSTALL_GMP} = true ]; then \
    # Install the PHP GMP extension
    apt-get -yqq install php${LARADOCK_PHP_VERSION}-gmp; \
    fi; \
    ###########################################################################
    # GnuPG:
    ###########################################################################
    if [ ${INSTALL_GNUPG} = true ]; then \
    apt-get -yqq install php${LARADOCK_PHP_VERSION}-gnupg; \
    fi; \
    ###########################################################################
    # SSH2:
    ###########################################################################
    if [ ${INSTALL_SSH2} = true ]; then \
    # Install the PHP SSH2 extension
    apt-get -yqq install libssh2-1-dev php${LARADOCK_PHP_VERSION}-ssh2; \
    fi; \
    ###########################################################################
    # SOAP:
    ###########################################################################
    if [ ${INSTALL_SOAP} = true ]; then \
    # Install the PHP SOAP extension
    apt-get -yqq install libxml2-dev php${LARADOCK_PHP_VERSION}-soap; \
    fi; \
    ###########################################################################
    # XSL:
    ###########################################################################
    if [ ${INSTALL_XSL} = true ]; then \
    # Install the PHP XSL extension
    apt-get -yqq install libxslt-dev php${LARADOCK_PHP_VERSION}-xsl; \
    fi

###########################################################################

ARG INSTALL_LDAP=false
ARG INSTALL_SMB=false
ARG INSTALL_IMAP=false
ARG INSTALL_SUBVERSION=false

RUN set -eux; \
    ###########################################################################
    # LDAP:
    ###########################################################################
    if [ ${INSTALL_LDAP} = true ]; then \
    apt-get install -yqq libldap2-dev php${LARADOCK_PHP_VERSION}-ldap; \
    fi; \
    ###########################################################################
    # SMB:
    ###########################################################################
    if [ ${INSTALL_SMB} = true ]; then \
    apt-get install -yqq smbclient php-smbclient coreutils; \
    fi; \
    ###########################################################################
    # IMAP:
    ###########################################################################
    if [ ${INSTALL_IMAP} = true ]; then \
    apt-get install -yqq php${LARADOCK_PHP_VERSION}-imap; \
    fi; \
    ###########################################################################
    # Subversion:
    ###########################################################################
    if [ ${INSTALL_SUBVERSION} = true ]; then \
    apt-get install -yqq subversion; \
    fi

###########################################################################
# xDebug:
###########################################################################

USER root

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Install the xdebug extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    pecl install xdebug-3.3.0; \
    else \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    pecl install xdebug-2.5.5; \
    else \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
    pecl install xdebug-2.9.0; \
    else \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ]; then \
    pecl install xdebug-2.9.8; \
    else \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
    pecl install xdebug-2.9.8; \
    else \
    #pecl install xdebug; \
    echo "xDebug 3 required, not supported."; \
    fi \
    fi \
    fi \
    fi \
    fi && \
    echo "zend_extension=xdebug.so" >> /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-xdebug.ini \
    ;fi

# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini

RUN if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    sed -i "s/xdebug.remote_host=/xdebug.client_host=/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_connect_back=0/xdebug.discover_client_host=false/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_port=9000/xdebug.client_port=9003/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.profiler_enable=0/; xdebug.profiler_enable=0/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.profiler_output_dir=/xdebug.output_dir=/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_mode=req/; xdebug.remote_mode=req/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_autostart=0/xdebug.start_with_request=yes/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.mode=debug/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini \
    ;else \
    sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini \
    ;fi
RUN sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/xdebug.ini


###########################################################################
# Phpdbg:
###########################################################################

USER root

ARG INSTALL_PHPDBG=false

RUN if [ ${INSTALL_PHPDBG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y --force-yes php${LARADOCK_PHP_VERSION}-phpdbg \
    ;fi


###########################################################################
# ssh:
###########################################################################

ARG INSTALL_WORKSPACE_SSH=false

COPY insecure_id_rsa /tmp/id_rsa
COPY insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
    rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
    && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
    && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
    && rm -f /tmp/id_rsa* \
    && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
    && cp -rf /root/.ssh /home/laradock \
    && chown -R laradock:laradock /home/laradock/.ssh \
    ;fi

###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    pecl install mongo && \
    echo "extension=mongo.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/mongo.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/mongo.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-mongo.ini \
    ;else \
    pecl install mongodb && \
    echo "extension=mongodb.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/mongodb.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/mongodb.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-mongodb.ini \
    ;fi \
    ;fi

###########################################################################
# AMQP:
###########################################################################

ARG INSTALL_AMQP=false

RUN if [ ${INSTALL_AMQP} = true ]; then \
    apt-get install -yqq librabbitmq-dev && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    printf "\n" | pecl install amqp-1.11.0beta; \
    else \
    printf "\n" | pecl install amqp; \
    fi && \
    echo "extension=amqp.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/amqp.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/amqp.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-amqp.ini \
    ;fi


###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    apt-get install -yqq php${LARADOCK_PHP_VERSION}-redis \
    ;fi

###########################################################################
# Swoole EXTENSION
###########################################################################

# ARG INSTALL_SWOOLE=false

# RUN set -eux; \
#   if [ ${INSTALL_SWOOLE} = true ]; then \
#   # Install Php Swoole Extension
#   if   [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "50600" ]; then \
#   echo '' | pecl -q install swoole-2.0.10; \
#   elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70000" ]; then \
#   echo '' | pecl -q install swoole-4.3.5; \
#   elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70100" ]; then \
#   echo '' | pecl -q install swoole-4.5.11; \
#   else \
#   printf "no\nyes\nyes\nno\nyes\nyes" | pecl -q install swoole-4.8.5; \
#   fi; \
#   echo "extension=swoole.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini; \
#   ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-swoole.ini; \
#   php -m | grep -q 'swoole'; \
#   fi

###########################################################################
# Swoole EXTENSION
###########################################################################

ARG INSTALL_SWOOLE=false

RUN set -eux; \
    if [ ${INSTALL_SWOOLE} = true ]; then \
    # Install Php Swoole Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    echo '' | pecl -q install swoole-2.0.10; \
    elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
    echo '' | pecl -q install swoole-4.3.5; \
    elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ]; then \
    echo '' | pecl -q install swoole-4.5.11; \
    elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
    echo '' | pecl -q install swoole-4.8.12; \
    else \
    echo '' | pecl -q install swoole; \
    fi; \
    echo "extension=swoole.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini; \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-swoole.ini; \
    php -m | grep -q 'swoole'; \
    fi


###########################################################################
# Inotify EXTENSION:
###########################################################################

ARG INSTALL_INOTIFY=false

RUN if [ ${INSTALL_INOTIFY} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    pecl -q install inotify-0.1.6 && \
    echo "extension=inotify.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/inotify.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/inotify.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-inotify.ini; \
    else \
    pecl -q install inotify && \
    echo "extension=inotify.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/inotify.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/inotify.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-inotify.ini \
    ;fi \
    ;fi

###########################################################################
# AST EXTENSION
###########################################################################

ARG INSTALL_AST=false
ARG AST_VERSION=1.0.10
ENV AST_VERSION ${AST_VERSION}

RUN if [ ${INSTALL_AST} = true ]; then \
    # AST extension requires PHP 7.0.0 or newer
    if [ $(php -r "echo PHP_MAJOR_VERSION;") != "5" ]; then \
    # Install AST extension
    printf "\n" | pecl -q install ast-${AST_VERSION} && \
    echo "extension=ast.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/ast.ini && \
    phpenmod -v ${LARADOCK_PHP_VERSION} -s cli ast \
    ;fi \
    ;fi


###########################################################################
# PHP OCI8:
###########################################################################

USER root
ARG INSTALL_OCI8=false
ARG ORACLE_INSTANT_CLIENT_MIRROR=https://github.com/diogomascarenha/oracle-instantclient/raw/master/

ENV LD_LIBRARY_PATH="/opt/oracle/instantclient_12_1"
ENV OCI_HOME="/opt/oracle/instantclient_12_1"
ENV OCI_LIB_DIR="/opt/oracle/instantclient_12_1"
ENV OCI_INCLUDE_DIR="/opt/oracle/instantclient_12_1/sdk/include"
ENV OCI_VERSION=12

RUN if [ ${INSTALL_OCI8} = true ]; then \
    # Install wget
    apt-get update && apt-get install --no-install-recommends -y wget \
    # Install Oracle Instantclient
    && mkdir /opt/oracle \
    && cd /opt/oracle \
    && wget ${ORACLE_INSTANT_CLIENT_MIRROR}instantclient-basic-linux.x64-12.1.0.2.0.zip \
    && wget ${ORACLE_INSTANT_CLIENT_MIRROR}instantclient-sdk-linux.x64-12.1.0.2.0.zip \
    && unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
    && rm -rf /opt/oracle/*.zip \
    # Install PHP extensions deps
    && apt-get update \
    && apt-get install --no-install-recommends -y \
    libaio-dev && \
    # Install PHP extensions
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.0.10; \
    else \
    echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.2.0; \
    fi \
    && echo "extension=oci8.so" >> /etc/php/${LARADOCK_PHP_VERSION}/cli/php.ini \
    && php -m | grep -q 'oci8' \
    ;fi


###########################################################################
# Prestissimo:
###########################################################################
ARG INSTALL_PRESTISSIMO=false

RUN if [ ${INSTALL_PRESTISSIMO} = true ]; then \
    if [ $(php -r "echo COMPOSER_VERSION;") = "1" ]; then \
    # Install Prestissimo
    composer global require "hirak/prestissimo" \
    ;fi \
    ;fi


###########################################################################
# SQL SERVER:
###########################################################################

ARG INSTALL_MSSQL=false

RUN set -eux; \
    if [ ${INSTALL_MSSQL} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    apt-get install -yqq php5.6-sybase freetds-bin freetds-common libsybdb5 \
    && php -m | grep -q 'mssql' \
    && php -m | grep -q 'pdo_dblib' \
    ;else \
    ###########################################################################
    #  The following steps were taken from
    #  https://github.com/Microsoft/msphpsql/wiki/Install-and-configuration
    ###########################################################################
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && \
    curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list > /etc/apt/sources.list.d/mssql-release.list && \
    apt-get update -yqq && \
    ACCEPT_EULA=Y apt-get install -yqq msodbcsql17 mssql-tools unixodbc unixodbc-dev libgss3 odbcinst locales && \
    ln -sfn /opt/mssql-tools/bin/sqlcmd /usr/bin/sqlcmd && \
    ln -sfn /opt/mssql-tools/bin/bcp /usr/bin/bcp && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "2" ]; then \
    pecl install pdo_sqlsrv-5.8.1 sqlsrv-5.8.1 \
    ;elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ]; then \
    pecl install pdo_sqlsrv-5.6.1 sqlsrv-5.6.1 \
    ;elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
    pecl install pdo_sqlsrv-5.3.0 sqlsrv-5.3.0 \
    ;else \
    pecl install pdo_sqlsrv sqlsrv \
    ;fi && \
    echo "extension=sqlsrv.so"     > /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-sqlsrv.ini && \
    echo "extension=pdo_sqlsrv.so" > /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-pdo_sqlsrv.ini && \
    php -m | grep -q 'sqlsrv' && \
    php -m | grep -q 'pdo_sqlsrv' \
    ;fi \
    ;fi


###########################################################################
# PYTHON3:
###########################################################################

ARG INSTALL_PYTHON3=false

# RUN if [ ${INSTALL_PYTHON3} = true ]; then \
#   apt-get -y install python3 python3-dev build-essential  \
#   && curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py  \
#   && python3 get-pip.py  \
#   && rm get-pip.py  \
#   && python3 -m pip install -i http://mirrors.aliyun.com/pypi/simple/ --upgrade --force-reinstall pip  \
#   && python3 -m pip install -i http://mirrors.aliyun.com/pypi/simple/ --upgrade virtualenv \
#   ;fi

RUN if [ ${INSTALL_PYTHON3} = true ]; then \
    apt-get -y install python3 python3-dev python3-pip build-essential  \
    && python3 -m pip install -i https://mirrors.aliyun.com/pypi/simple/ --upgrade --force-reinstall pip  \
    && python3 -m pip install -i https://mirrors.aliyun.com/pypi/simple/ --upgrade virtualenv \
    ;fi

###########################################################################
# SUPERVISOR:
###########################################################################
ARG INSTALL_SUPERVISOR=false

RUN if [ ${INSTALL_SUPERVISOR} = true ]; then \
    if [ ${INSTALL_PYTHON3} = true ]; then \
    python3 -m pip install -i https://mirrors.aliyun.com/pypi/simple/ --upgrade supervisor && \
    echo_supervisord_conf > /etc/supervisord.conf && \
    sed -i 's/\;\[include\]/\[include\]/g' /etc/supervisord.conf && \
    sed -i 's/\;files\s.*/files = supervisord.d\/*.conf/g' /etc/supervisord.conf \
    ;fi \
    ;fi

USER laradock


###########################################################################
USER root

ARG INSTALL_MYSQL_CLIENT=false
ARG INSTALL_PING=false
ARG INSTALL_SSHPASS=false
ARG INSTALL_DOCKER_CLIENT=false

RUN set -eux; \
    ###########################################################################
    # MySQL Client:
    ###########################################################################
    if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
    apt-get -yqq install mysql-client; \
    fi; \
    ###########################################################################
    # ping:
    ###########################################################################
    if [ ${INSTALL_PING} = true ]; then \
    apt-get -yqq install inetutils-ping; \
    fi; \
    ###########################################################################
    # sshpass:
    ###########################################################################
    if [ ${INSTALL_SSHPASS} = true ]; then \
    apt-get -yqq install sshpass; \
    fi; \
    ###########################################################################
    # Docker Client:
    ###########################################################################
    if [ ${INSTALL_DOCKER_CLIENT} = true ]; then \
    curl -sS https://download.docker.com/linux/static/stable/x86_64/docker-20.10.3.tgz -o /tmp/docker.tar.gz; \
    tar -xzf /tmp/docker.tar.gz -C /tmp/; \
    cp /tmp/docker/docker* /usr/local/bin; \
    chmod +x /usr/local/bin/docker*; \
    fi

###########################################################################
USER root

ARG INSTALL_YAML=false
ARG INSTALL_RDKAFKA=false
ARG INSTALL_FFMPEG=false

RUN set -eux; \
    ###########################################################################
    # YAML: extension for PHP-CLI
    ###########################################################################
    if [ ${INSTALL_YAML} = true ]; then \
    apt-get install -yqq libyaml-dev; \
    if   [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    echo '' | pecl install -a yaml-1.3.2; \
    elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
    echo '' | pecl install yaml-2.0.4; \
    else \
    echo '' | pecl install yaml; \
    fi; \
    echo "extension=yaml.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/yaml.ini; \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/yaml.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/35-yaml.ini; \
    fi; \
    ###########################################################################
    # RDKAFKA:
    ###########################################################################
    if [ ${INSTALL_RDKAFKA} = true ]; then \
    apt-get install -yqq librdkafka-dev; \
    pecl install rdkafka; \
    echo "extension=rdkafka.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/rdkafka.ini; \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/rdkafka.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-rdkafka.ini; \
    php -m | grep -q 'rdkafka'; \
    fi; \
    ###########################################################################
    # FFMpeg:
    ###########################################################################
    if [ ${INSTALL_FFMPEG} = true ]; then \
    apt-get -yqq install ffmpeg; \
    fi


#####################################
# wkhtmltopdf:
#####################################

USER root

ARG INSTALL_WKHTMLTOPDF=false

RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    apt-get install -y \
    libxrender1 \
    libfontconfig1 \
    libx11-dev \
    libjpeg62 \
    libxtst6 \
    fontconfig \
    libjpeg-turbo8-dev \
    xfonts-base \
    xfonts-75dpi \
    wget \
    && wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.bionic_amd64.deb \
    && dpkg -i wkhtmltox_0.12.6-1.bionic_amd64.deb \
    && apt -f install \
    ;fi


###########################################################################
# Bash Git Prompt
###########################################################################

ARG INSTALL_GIT_PROMPT=false

COPY git-prompt.sh /tmp/git-prompt

RUN if [ ${INSTALL_GIT_PROMPT} = true ]; then \
    git clone https://github.com/magicmonty/bash-git-prompt.git /root/.bash-git-prompt --depth=1 && \
    cat /tmp/git-prompt >> /root/.bashrc && \
    rm /tmp/git-prompt \
    ;fi

###########################################################################
# XMLRPC:
###########################################################################

ARG INSTALL_XMLRPC=false

RUN if [ ${INSTALL_XMLRPC} = true ]; then \
    apt-get install -yqq php${LARADOCK_PHP_VERSION}-xmlrpc \
    ;fi

###########################################################################
# Lnav:
###########################################################################

ARG INSTALL_LNAV=false

RUN if [ ${INSTALL_LNAV} = true ]; then \
    apt-get install -yqq lnav \
    ;fi


###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${LARADOCK_PHP_VERSION}."


###########################################################################
# PHP DECIMAL:
###########################################################################

USER root

ARG INSTALL_PHPDECIMAL=false

RUN if [ ${INSTALL_PHPDECIMAL} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    echo 'decimal not support PHP 5.6'; \
    else \
    apt-get install -yqq libmpdec-dev \
    && pecl install decimal \
    && echo "extension=decimal.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/decimal.ini \
    && ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/decimal.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-decimal.ini \
    && php -m | grep -q 'decimal' \
    ;fi \
    ;fi

###########################################################################
# zookeeper
###########################################################################
ARG INSTALL_ZOOKEEPER=false

RUN set -eux; \
    if [ ${INSTALL_ZOOKEEPER} = true ]; then \
    apt install -yqq libzookeeper-mt-dev; \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
    curl -L -o /tmp/php-zookeeper.tar.gz https://github.com/php-zookeeper/php-zookeeper/archive/master.tar.gz; \
    mkdir -p /tmp/php-zookeeper; \
    tar -C /tmp/php-zookeeper -zxvf /tmp/php-zookeeper.tar.gz --strip 1; \
    cd /tmp/php-zookeeper; \
    phpize && ./configure && make && make install;\
    else \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
    pecl install zookeeper-0.5.0; \
    else \
    pecl install zookeeper-0.7.2; \
    fi; \
    fi; \
    echo "extension=zookeeper.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/zookeeper.ini; \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/zookeeper.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-zookeeper.ini; \
    php -m | grep -q 'zookeeper'; \
    fi

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www
