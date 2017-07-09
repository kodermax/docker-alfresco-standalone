######################################################################
# Dockerfile
# Author: Maksim Karpychev
# Date:   07 July 2017
#
# Alfresco server docker image.
#
# Repository, Solr and Share all-in-one image prototype.
# This server expects a Postgres DB to be available.
#
#
######################################################################

FROM ubuntu:14.04

MAINTAINER Maksim Karpychev <kodermax@gmail.com>

ENV ALF_INSTALLER_BIN http://eu.dl.alfresco.com.s3.amazonaws.com/release/community/201704-build-00019/alfresco-community-installer-201704-linux-x64.bin
ENV ALF_INSTALL_OPTIONS_FILE install_options.txt
ENV ALF_INSTALLER_DIR /alfresco/installer
ENV ALFRESCO_HOME /opt/alfresco-one
ENV ALF_PROPS_FILE $ALFRESCO_HOME/tomcat/shared/classes/alfresco-global.properties


# Install OpenOffice requirements and other utils
RUN apt-get update && apt install -y \
    libc6 \
    libcups2 \
    libdbus-glib-1-2 \
    libfontconfig1 \
    libfreetype6 \
    libx11-6 \
    libxau6 \
    libxdmcp6 \
    libxext6 \
    libxinerama1 \
    unzip \
    vim

RUN mkdir -p $ALF_INSTALLER_DIR

# This really adds bloat, the installer could be deleted after it's used, but
# doing it with ADD or COPY results in a layer being added to the image. Only
# way I know of avoiding this at present is to use curl to download the installer from somewhere,
# run it and delete it, but all in a *single* RUN directive.
COPY $ALF_INSTALLER_BIN $ALF_INSTALLER_DIR/
COPY $ALF_INSTALL_OPTIONS_FILE $ALF_INSTALLER_DIR/

WORKDIR $ALF_INSTALLER_DIR


# Blimey, chmod results in a 829.4MB addition if left to its own devices.
# Removing the ALF_INSTALLER_BIN at the end of this command doesn't stop the COPY above from creating its 829.4MB
# layer, but it does stop the chmod from resulting in a further 829.4MB addition (since the file isn't there
# to commit the change against)
RUN chmod +x $ALF_INSTALLER_BIN && ./$ALF_INSTALLER_BIN --optionfile $ALF_INSTALL_OPTIONS_FILE && rm $ALF_INSTALLER_BIN

EXPOSE 8080 7070 5701


WORKDIR $ALFRESCO_HOME


RUN mkdir /alfresco/data
RUN sed -i 's:^[ \t]*dir.root[ \t]*=\([ \t]*.*\)$:dir.root='/alfresco/data':' $ALF_PROPS_FILE

# Change dir.keystore to point to installed keystore - would be nicer to ADD the keystore
# to the data volume
RUN sed -i 's:^[ \t]*dir.keystore[ \t]*=\([ \t]*.*\)$:dir.keystore='${ALFRESCO_HOME}/alf_data/keystore':' $ALF_PROPS_FILE

# Ensure a log file exists ready for tail
RUN touch $ALFRESCO_HOME/alfresco.log

# Logging
ADD dev-log4j.properties $ALFRESCO_HOME/tomcat/shared/classes/alfresco/extension/

# Deploy custom war files
#ADD *.war $ALFRESCO_HOME/tomcat/webapps/

# Install a license file if one exists
#ADD *.lic $ALFRESCO_HOME/tomcat/shared/classes/alfresco/extension/license/

# Use instead of alfresco.sh -- but it will invoke it.
# This is only when starting the container, it is fine to run alfresco.sh to start and stop
# Alfresco once the container has been initialised.
ADD docker_start $ALFRESCO_HOME/
RUN chmod +x $ALFRESCO_HOME/docker_start

# Run the container's process!
CMD /bin/bash -c $ALFRESCO_HOME/docker_start