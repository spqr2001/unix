FROM {{ wildfly_source }} 

ADD war/helloworld.war /opt/jboss/wildfly/standalone/deployments/

ARG {{ wildfly_name }}  
ARG {{ wildfly_cluster_pw }}  

ENV {{ wildfly_name }}=${WILDFLY_PW}

ENTRYPOINT /opt/jboss/wildfly/bin/standalone.sh -b=0.0.0.0 -bmanagement=0.0.0.0 -Djboss.server.default.config=standalone-full-ha.xml -Djboss.node.name={{ wildfly_name }} -Djava.net.preferIPv4Stack=true -Djgroups.bind_addr={{ ansible_default_ipv4.address }} -Djboss.messaging.cluster.password={{ wildfly_cluster_pw }}
