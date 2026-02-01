# -------------------------------------------------
# BUILDER STAGE
# -------------------------------------------------
FROM registry.redhat.io/jboss-eap-8/eap81-openjdk21-builder-openshift-rhel9:latest AS builder

ENV GALLEON_PROVISION_FEATURE_PACKS=org.jboss.eap:wildfly-ee-galleon-pack,org.jboss.eap.cloud:eap-cloud-galleon-pack
ENV GALLEON_PROVISION_LAYERS cloud-default-config,messaging-activemq,datasources
ENV GALLEON_PROVISION_CHANNELS=org.jboss.eap.channels:eap-8.1

RUN /usr/local/s2i/assemble


# -------------------------------------------------
# RUNTIME STAGE
# -------------------------------------------------
FROM registry.redhat.io/jboss-eap-8/eap81-openjdk21-runtime-openshift-rhel9:latest

COPY --from=builder --chown=jboss:root $JBOSS_HOME $JBOSS_HOME


# -------------------------------------------------
# POSTGRESQL JDBC MODULE
# -------------------------------------------------
COPY postgresql-42.5.6.jar /tmp/postgresql.jar

RUN mkdir -p $JBOSS_HOME/modules/org/postgresql/main && \
    cp /tmp/postgresql.jar $JBOSS_HOME/modules/org/postgresql/main/postgresql.jar && \
    printf '<module xmlns="urn:jboss:module:1.5" name="org.postgresql">\n\
  <resources>\n\
    <resource-root path="postgresql.jar"/>\n\
  </resources>\n\
  <dependencies>\n\
    <module name="javax.api"/>\n\
    <module name="javax.transaction.api"/>\n\
  </dependencies>\n\
</module>' > $JBOSS_HOME/modules/org/postgresql/main/module.xml


# -------------------------------------------------
# SERVER CONFIGURATION (DS + RA + JMS)
# -------------------------------------------------
COPY --chown=jboss:root coolstore-config.cli /tmp/coolstore-config.cli

RUN $JBOSS_HOME/bin/jboss-cli.sh --file=/tmp/coolstore-config.cli


# -------------------------------------------------
# APPLICATION (DEPLOY LAST)
# -------------------------------------------------
COPY --chown=jboss:root ROOT.war $JBOSS_HOME/standalone/deployments/


# -------------------------------------------------
# PERMISSIONS
# -------------------------------------------------
RUN chmod -R ug+rwX $JBOSS_HOME
