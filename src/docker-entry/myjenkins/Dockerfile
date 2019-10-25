FROM jenkins/jenkins:alpine

ENV JAVA_OPTS=-Djenkins.install.runSetupWizard=false

COPY create-admin.groovy /usr/share/jenkins/ref/init.groovy.d/
