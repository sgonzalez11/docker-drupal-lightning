FROM httpd:2.4

ARG WEB_USER
ARG WEB_GROUP
ARG APACHE_ROOT_DIR

COPY ./conf/httpd-vhosts.conf /usr/local/apache2/conf/extra/httpd-vhosts.conf
COPY ./conf/httpd.conf /usr/local/apache2/conf/httpd.conf

RUN chgrp -R ${WEB_GROUP} ${APACHE_ROOT_DIR}/conf/httpd.conf \
 && chgrp -R ${WEB_GROUP} ${APACHE_ROOT_DIR}/conf/extra/httpd-vhosts.conf

RUN usermod -u 1000 ${WEB_USER} \
 && groupmod -g 1000 ${WEB_GROUP} \
 && chgrp -R ${WEB_GROUP} ${APACHE_ROOT_DIR}