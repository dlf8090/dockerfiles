ARG MYSQL_VERSION=5.7
FROM mysql:${MYSQL_VERSION}

LABEL maintainer="donglifan <donggefor2018@163.com>"

#####################################
# Set Timezone
#####################################

ARG TZ=PRC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && chown -R mysql:root /var/lib/mysql/

COPY my.cnf /etc/mysql/conf.d/my.cnf

RUN chmod 644 /etc/mysql/conf.d/my.cnf

EXPOSE 3306

CMD ["mysqld"]