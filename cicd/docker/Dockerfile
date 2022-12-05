FROM nexusprod.corp.intranet:4567/automation-ninjas/ctp/testauto-arcade/arcade:latest
ARG user=jenkins 
ARG group=jenkins
ARG uid=1341 
ARG gid=1341 
USER root 


SHELL ["/bin/bash", "-c"] 
WORKDIR /soatest 

 

# Install crontab, curl, vi, less, and ssmtp 
# RUN apt-get update && apt-get -y install cron && apt-get -y install curl && apt-get -y install vim && apt-get -y install less 
# Copy necessary files into image 
#COPY parasoft_soavirt_9.10.8_linux_x86_64.tar.gz soatest.tar.gz 
#COPY localsettings.properties localsettings.properties 
#COPY test_config.properties test_config.properties 

RUN tar xvf soatest.tar.gz 
RUN ./parasoft_soavirt_9.10_linux_x86_64_20191027.sh -l en --non-interactive 
RUN touch output.log 
RUN chmod -R 777 /soatest 

COPY . .  

# finally run the cron 
CMD tail -f output.log 
#CMD '/bootstrap.sh' 
