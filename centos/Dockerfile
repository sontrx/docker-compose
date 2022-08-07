FROM centos:7

ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

RUN yum check-update; \
  yum install -y initscripts openssh-server openssh-clients; \
  yum clean all

RUN ssh-keygen -A
RUN echo 'root:root' | chpasswd
EXPOSE 22
# ENTRYPOINT service ssh restart && bash
CMD ["/usr/sbin/sshd", "-D"]
