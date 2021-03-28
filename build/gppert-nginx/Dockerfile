# This image uses nvm and same base image as the worker image.
# This is done to ensures that node-sass binary remains common.
# node-sass is required to enable website theme feature used
# by Website Manager role in Frappe Framework
FROM python:3.7-slim-buster

ENV NVM_DIR=/root/.nvm
ENV NODE_VERSION=12.20.0
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"
RUN apt-get update -y \
    && apt-get install wget python3 git build-essential -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && wget https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh \
    && chmod +x install.sh \
    && ./install.sh \
    && . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION} \
    && nvm use v${NODE_VERSION} && npm install -g yarn

WORKDIR /home/frappe/frappe-bench
RUN mkdir -p /home/frappe/frappe-bench/sites \
    && echo "frappe\nerpnext\ngppert" > /home/frappe/frappe-bench/sites/apps.txt 

# ssh 
RUN apt-get install openssh-client
RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

ARG GIT_BRANCH=gppert
RUN --mount=type=ssh mkdir -p apps sites/assets/css  \
    && cd apps \
    && git clone --depth 1 https://github.com/Havenir/frappe.git --branch $GIT_BRANCH \
    && git clone --depth 1 https://github.com/Havenir/erpnext.git --branch $GIT_BRANCH \
    && git clone --depth 1 git@github.com:Havenir/gppert.git --branch main gppert

# for frappe
RUN cd /home/frappe/frappe-bench/apps/frappe \
    && yarn \
    && yarn run production \
    && yarn install --production=true

RUN node --version \
    && npm --version \
    && yarn --version

RUN git clone --depth 1 https://github.com/frappe/bench /tmp/bench \
    && mkdir -p /var/www/error_pages \
    && cp -r /tmp/bench/bench/config/templates /var/www/error_pages

# for frappe
RUN mkdir -p /home/frappe/frappe-bench/sites/assets/frappe/ \
    && cp -R /home/frappe/frappe-bench/apps/frappe/frappe/public/* /home/frappe/frappe-bench/sites/assets/frappe \
    && cp -R /home/frappe/frappe-bench/apps/frappe/node_modules /home/frappe/frappe-bench/sites/assets/frappe/

# for erpnext
RUN mkdir -p /home/frappe/frappe-bench/sites/assets/erpnext/ \
    && cp -R /home/frappe/frappe-bench/apps/erpnext/erpnext/public/* /home/frappe/frappe-bench/sites/assets/erpnext
    
# for gppert
RUN mkdir -p /home/frappe/frappe-bench/sites/assets/gppert/ \
    && cp -R /home/frappe/frappe-bench/apps/gppert/gppert/public/* /home/frappe/frappe-bench/sites/assets/gppert
    
FROM nginx:latest
COPY --from=0 /home/frappe/frappe-bench/sites /var/www/html/
COPY --from=0 /var/www/error_pages /var/www/
COPY build/common/nginx-default.conf.template /etc/nginx/conf.d/default.conf.template
COPY build/gppert-nginx/docker-entrypoint.sh /

RUN apt-get update && apt-get install -y rsync && apt-get clean \
    && echo '#!/bin/bash' > /rsync \
    && chmod +x /rsync

VOLUME [ "/assets" ]

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
