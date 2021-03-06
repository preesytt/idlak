FROM centos:7

ENV CPU_CORE 4

# Activate systemd. Taken from https://hub.docker.com/_/centos
ENV container docker
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
VOLUME [ "/sys/fs/cgroup" ]

RUN yum update -y
RUN yum groupinstall -y "Development Tools" "System Tools"
RUN yum install -y \
    git bzip2 wget subversion which sox \
    gcc-c++ make automake autoconf zlib-devel atlas-static \
     python python3-devel cmake tcsh pcre-devel
RUN yum remove -y swig
RUN yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/swig3-3.0.12-17.el7.x86_64.rpm

WORKDIR /usr/local/
# Use the newest idlak version
RUN git clone https://github.com/Idlak/idlak.git
WORKDIR /usr/local/idlak
RUN git checkout server-docker
RUN git pull

WORKDIR /usr/local/idlak/tools
RUN mkdir python
RUN touch python/.use_default_python
RUN extras/check_dependencies.sh
RUN make -j $CPU_CORE
RUN ./install_idlak.sh

WORKDIR /usr/local/idlak/src
ENV SWIG=/usr/bin/swig
RUN ./configure --shared
RUN make clean
RUN make depend -j $CPU_CORE
RUN make -j $CPU_CORE
RUN make ext -j $CPU_CORE

WORKDIR /usr/local/idlak/idlak-server
RUN python3 -m pip install -r requirements.txt

EXPOSE 80

# setup nginx
RUN yum install -y epel-release
RUN yum install -y nginx

# setup idlak service
RUN cp docker/wsgi.ini ./.
RUN cp docker/idlak.service /etc/systemd/system/.
RUN cp docker/idlak.conf /etc/nginx/conf.d/idlak.conf
RUN systemctl enable idlak

# continue setting up nginx
RUN usermod -a -G root nginx
RUN chmod 710 /usr/local/idlak
RUN systemctl enable nginx

CMD ["/usr/sbin/init"]

