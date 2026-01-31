FROM image-registry.openshift-image-registry.svc:5000/openshift/jboss-eap74-openjdk11-openshift:latest

USER root

# PostgreSQL module
RUN mkdir -p /opt/eap/modules/system/layers/base/org/postgresql/main
COPY postgresql-42.5.6.jar /opt/eap/modules/system/layers/base/org/postgresql/main/
COPY module.xml /opt/eap/modules/system/layers/base/org/postgresql/main/

# CLI script
COPY datasource-jms.cli /tmp/datasource-jms.cli
COPY jms.cli /opt/eap/extensions/jms.cli

# IMPORTANT: do NOT disable JMS broker during build
RUN /opt/eap/bin/jboss-cli.sh --file=/tmp/datasource-jms.cli

# Permissions for OpenShift
RUN chown -R 185:0 /opt/eap && chmod -R g+rwX /opt/eap

# Copy code
COPY ROOT.war /opt/eap/standalone/deployments/ROOT.war

USER 185
