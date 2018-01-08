FROM mysql:5.6
LABEL io.github.islandora-collaboration-group.name="isle-mysql" \
      io.github.islandora-collaboration-group.service="MySQLd" \
      io.github.islandora-collaboration-group.service.version="5.6" \
      io.github.islandora-collaboration-group.vcs-url="git@github.com:Islandora-Collaboration-Group/ISLE.git" \
      io.github.islandora-collaboration-group.vendor="Islandora Collaboration Group (ICG) - islandora-consortium-group@googlegroups.com" \
      io.github.islandora-collaboration-group.description="Fedora Repository and Dependencies" \
      io.github.islandora-collaboration-group.license="Apache-2.0" \
      io.github.islandora-collaboration-group.version="Alpha-2.0.1" 

COPY my.cnf /etc/alternatives/my.cnf
COPY init_dbs/ /docker-entrypoint-initdb.d/

EXPOSE 3306
CMD ["mysqld"]
