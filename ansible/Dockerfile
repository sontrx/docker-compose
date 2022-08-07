FROM centos:7

ENV ANSIBLE_CONFIG ./ansible.cfg

RUN yum check-update; \
  yum install -y gcc epel-release openssh-server openssh-clients; \
  yum install -y ansible; \
  yum clean all; \
  /usr/bin/ssh-keygen -A

# RUN  /usr/bin/ssh-keygen -A
COPY ./.ssh /root/.ssh

RUN chmod 600 /root/.ssh/*
