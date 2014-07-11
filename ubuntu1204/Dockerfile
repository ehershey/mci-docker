# Docker file to install MCI dependencies into a base Ubuntu 12.04 image
#
# A separate file with private information can be appended to this and used 
# with a command like:
# cat Dockerfile PrivateDockerfile | docker build -t mci:ubuntu1204 -
# 
#
FROM ubuntu:12.04
MAINTAINER Ernie Hershey <ernie@mongodb.com>

# START Docker additions
#

# To route syslog to the outside host
#
VOLUME /dev:/hostdev
RUN ln -s /hostdev/log /dev/log

# Install apps that would be there by default
#
RUN apt-get update
RUN apt-get install --force-yes --assume-yes openssh-server sudo python-software-properties

# Add Ubuntu user 
#
RUN useradd -m ubuntu
RUN mkdir -p /home/ubuntu/.ssh

# Docker setup for MCI
#
RUN mkdir -p /data
RUN echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCKtThuRvdQtajq5wd+2Jfc4r3u1o+kvfbtFB4bbc3t/5pqse0ptAMS3eoitWd4VZTYb1pxJUaoKSGCsF8HeNfFq2INaCo3gbnEnBnqWnYAvBBNNxfB/VSxSIuZ6q2AAk9kQfhPUXnCEGVcbzU3HG2ELL0Xg6L5fmPnGvqFLXDmtqwAdeRzXvzvevv7uWkWtoHehNN8jGR2NqxnghlBiYntcdF+ajMzPYGorOYct6aixR8ye4f9srTS4U+nJxf4qkJs1+mgr7l+WrwUePWdM0MWeYehlov+SiJlaFRzdpeoNcNEmKdx6Gkzb9y8Vp2Ia95zmgfGzl21U9Sil316J mci-ernie" >> /home/ubuntu/.ssh/authorized_keys

RUN echo "github.com,207.97.227.239 ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==" | tee -a /home/ubuntu/.ssh/known_hosts
RUN echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDOKHeJ/AfWWTnwhiZYSPQ4HRSe84sf4qUgyAGGU+rJkq6Pwuk064/wh+D2vx1dOMQMETAcV4xB3AYw3S04ZHKwz61SLJSyjvJ3NItU7SKa8dNxr1EJywfk63tTAudleH4DjAFzHdRMqZT1penOhEJtKruyzxVuSpwFjkjJY/3vhawu9oNiolGhGvm/QzbxF8CjhWsLk/oAT8poWCsbvz5WlU6ttSMc6dkPLIB4kj0nlq8QDHdK6GnB9DRIGActa6fJaDRxGqgmnYIk5W44nFTGqIcfvqBC7LM83rnVeaseVWytSlbAQL11b+ZDw+UaQeT4CZ9Ft9Wi2MNuqU+NHu4f slave@10gen.com" | tee -a /home/ubuntu/.ssh/id_rsa.pub
RUN chmod 600 /home/ubuntu/.ssh/*
RUN chown -R ubuntu /data
# 
# END Docker additions


RUN set -o errexit
RUN set -o verbose
# Prevent apt-get from asking questions using whiptail or dialog
# 
RUN export DEBIAN_FRONTEND=noninteractive
# wait up to 30 seconds for sources list to be updated
# Commented out for docker
# RUN SOURCELIST_RECENT_MAX_MINUTES=90
# RUN SOURCELIST_RECENT_CHECK_TIMEOUT=30
# RUN now=`date +%s` 
# RUN while [ ! "`find /etc/apt/sources.list -mmin -$SOURCELIST_RECENT_MAX_MINUTES`" -a "`expr \"\`date +%s\`\" - $now`" -lt $SOURCELIST_RECENT_CHECK_TIMEOUT ]  
# RUN do 
# RUN         sleep 1 
# RUN done
RUN apt-get update || true
RUN yes | sudo add-apt-repository ppa:duh/golang
RUN apt-get install --force-yes --assume-yes g++ python-dev cyrus-sasl2-dbg cyrus-sasl2-mit-dbg libsasl2-2 libsasl2-dev libsasl2-modules libsasl2-modules-gssapi-mit krb5-user git python-pip libkrb5-dev libsnmp-dev libboost-thread-dev libboost-filesystem-dev libboost-program-options-dev xfsprogs mdadm=3.2.3-2ubuntu1 postfix=2.9.1-4 golang make
RUN pip install pymongo simples3 cpplint

# scons 2.3 required for enterprise builds
#
RUN wget http://downloads.sourceforge.net/project/scons/scons/2.3.0/scons-2.3.0.tar.gz
RUN tar zxvf scons-2.3.0.tar.gz 
RUN python scons-2.3.0/setup.py install

#umount /mnt || true
#umount /dev/xvdd || true
#umount /dev/xvde || true
#udevadm control --stop-exec-queue
#yes | mdadm --create /dev/md0 --level=0 -c256 --raid-devices=2 /dev/xvdd /dev/xvde
#udevadm control --start-exec-queue
#blockdev --setra 32 /dev/md0
#/sbin/mkfs.xfs -f /dev/md0
RUN mkdir -p /data
#echo "/dev/md0 /data auto noatime 0 0" | tee -a /etc/fstab
#mount /data

RUN echo "ubuntu hard nofile 64000" | tee -a /etc/security/limits.conf
RUN echo "ulimit -n 64000" >> /home/ubuntu/.bashrc
RUN echo "ulimit -n 64000" >> /home/ubuntu/.bash_profile
RUN chown -R ubuntu:ubuntu /data
RUN chown -R ubuntu:ubuntu /home/ubuntu/.ssh
