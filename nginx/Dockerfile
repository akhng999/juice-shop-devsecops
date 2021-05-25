# For Debian 10:
FROM debian:buster

# Download certificate and key from the customer portal (https://my.f5.com)
# and copy to the build context:
COPY nginx-repo.crt nginx-repo.key /etc/ssl/nginx/

# Install prerequisite packages:
RUN apt-get update && apt-get install -y apt-transport-https lsb-release ca-certificates wget gnupg2

# Download and add the NGINX signing key:
RUN wget https://cs.nginx.com/static/keys/nginx_signing.key && apt-key add nginx_signing.key

# Add NGINX Plus repository:
RUN printf "deb https://plus-pkgs.nginx.com/debian `lsb_release -cs` nginx-plus\n" | tee /etc/apt/sources.list.d/nginx-plus.list

# Download the apt configuration to `/etc/apt/apt.conf.d`:
RUN wget -P /etc/apt/apt.conf.d https://cs.nginx.com/static/files/90nginx

# Update the repository and install the most recent version of the NGINX App Protect package (which includes NGINX Plus):
RUN apt-get update && apt-get install -y app-protect

#############################################################################
# AppProtect 
#############################################################################
# Add NGINX App Protect Security Updates repository:
RUN printf "deb https://app-protect-security-updates.nginx.com/debian/ `lsb_release -cs` nginx-plus\n" | tee /etc/apt/sources.list.d/app-protect-security-updates.list

# Download and add the NGINX App Protect signatures signing key:
RUN wget https://cs.nginx.com/static/keys/app-protect-security-updates.key \
    && apt-key add app-protect-security-updates.key

# Download the apt configuration to /etc/apt/apt.conf.d:
RUN wget -P /etc/apt/apt.conf.d https://cs.nginx.com/static/files/90app-protect-security-updates

# Update attack signatures:
RUN apt-get update \
    && apt-get -y install app-protect-attack-signatures

# Update Threat campaigns:
RUN apt-get -y install app-protect-threat-campaigns

###########################################################################

# Remove nginx repository key/cert from docker
RUN rm -rf /etc/ssl/nginx

# Forward request logs to Docker log collector:
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

# Copy configuration files:
COPY config/nginx.conf /etc/nginx/
COPY config/*.json /etc/nginx/conf.d/
COPY config/default.conf /etc/nginx/conf.d/
COPY entrypoint.sh  /root/

CMD ["sh", "/root/entrypoint.sh"]