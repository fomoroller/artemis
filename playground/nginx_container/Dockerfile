# Copyright 2026 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM nginx:1.27-alpine

# eMudhra CA distrust policy (2026-10-21)
RUN apk add --no-cache --upgrade ca-certificates \
    && sed -i -E 's#^mozilla/(Comodo_AAA_Services_root|emSign_ECC_Root_CA_-_G3)\.crt$#!&#' /etc/ca-certificates.conf \
    && update-ca-certificates --fresh
ENV SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt \
    SSL_CERT_DIR=/opt/emudhra-empty-cert-directory \
    REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt \
    CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt \
    AWS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt

# Install openssl for auto-generating self-signed dev certificates and curl for healthchecks
RUN apk add --no-cache openssl curl bash

# Create directory structure
RUN mkdir -p /etc/nginx/conf.d \
             /etc/nginx/includes \
             /etc/nginx/certs \
             /usr/share/nginx/html

# Copy main Nginx configuration
COPY conf/nginx.conf /etc/nginx/nginx.conf

# Clear base image default configs and copy custom configs
RUN rm -rf /etc/nginx/conf.d/*
COPY conf/conf.d/ /etc/nginx/conf.d/
COPY conf/includes/ /etc/nginx/includes/

# Copy custom error templates and fallback static files
COPY html/ /usr/share/nginx/html/

# Copy entrypoint script
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

# Expose standard HTTP and HTTPS ports
EXPOSE 80 443

# Healthcheck to ensure Nginx is responding
HEALTHCHECK --interval=15s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -k -f https://localhost/healthz || curl -f http://localhost:80/healthz || exit 1

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
