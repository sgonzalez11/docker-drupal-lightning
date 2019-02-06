FROM percona

ARG MYSQL_CONTAINER_USER
ARG MYSQL_CONTAINER_GROUP

COPY ./conf/local.cnf /etc/my.cnf.d/local.cnf

# RUN usermod -u 1000 ${MYSQL_CONTAINER_USER} \
# && groupmod -g 1000 ${MYSQL_CONTAINER_GROUP}
