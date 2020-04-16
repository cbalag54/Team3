# Team3 Project

Gitlab+Sonarqube installation

Ensure external url consist of <host_machine_ip:host_machine_port> of your host machine.

GITLAB_OMNIBUS_CONFIG: |
  external_url 'http://<host_machine_ip>:<host_machine_port>'

Gitlab ports are published at 80:80,443:443, & 22:22
SonarQube port is published at 9000:9000

All docker volumes are in default location: /var/lib/docker/volumes/

NodeJS needs to be installed for SonarQube to scan Javascript codes
