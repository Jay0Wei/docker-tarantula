FROM centos:centos6
MAINTAINER ome-devel@lists.openmicroscopy.org.uk

RUN yum install -y \
	epel-release && \
	yum install -y make gcc readline-devel zlib-devel openssl-devel \
	libyaml redhat-lsb

RUN gpg2 --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3 && \
	curl -L https://get.rvm.io | bash -s stable
RUN bash -lc "source /usr/local/rvm/scripts/rvm && \
	rvm install 1.9.3 && \
	echo 'gem: --no-rdoc --no-ri' > /etc/gemrc && \
	gem install rails -v 3.2.21"

ENV TARANTULA_VERSION 2014.26.1

# Utilities
# Runtime requirements for Tarantula
# Requirements for bundle install
# Requirements for compiling passenger
RUN yum install -y unzip supervisor \
	mysql-devel httpd memcached cronie \
	libxml2-devel \
	curl-devel httpd-devel apr-devel apr-util-devel

RUN mkdir -p /opt/tarantula \
	/data/attachment_files /data/log /data/tmp /data/supervisor
ADD tarantula-setup.sh /opt/tarantula/
ADD apache-tarantula.conf /opt/tarantula/
ADD database.yml /opt/tarantula/
ADD https://github.com/prove/tarantula/archive/$TARANTULA_VERSION.zip /opt/tarantula/
ADD supervisord.conf /etc/supervisord.conf

WORKDIR /opt/tarantula

RUN unzip $TARANTULA_VERSION.zip && \
	ln -s tarantula-$TARANTULA_VERSION rails

RUN cd rails && \
	rm -rf attachment_files log tmp && \
	ln -s /data/attachment_files /data/log /data/tmp . && \
	touch log/production.log && \
	bash -lc "bundle install --deployment" && \
	cp /opt/tarantula/rails/config/crontab /etc/cron.d/tarantula

RUN bash -lc "gem install passenger && \
	passenger-install-apache2-module -a && \
	passenger-install-apache2-module --snippet > \
		/etc/httpd/conf.d/tarantula.conf && \
	cat /opt/tarantula/apache-tarantula.conf >> \
		/etc/httpd/conf.d/tarantula.conf"

# install.sh sets the entire tarantula install to be owned by apache
# in practice it seems to be sufficient to make just config.ru owned
#RUN chown -R apache:apache /opt/tarantula/
RUN chown -R apache:apache rails/config.ru \
	/data/attachment_files /data/log /data/tmp

CMD ["/usr/bin/supervisord"]

