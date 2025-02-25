# Use a GDAL image from OSGeo as the base
ARG GDAL_VERSION=ubuntu-small-3.10.2
FROM ghcr.io/osgeo/gdal:${GDAL_VERSION}

# Set noninteractive mode for apt-get to avoid prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies needed for building and running ODC
RUN apt-get update && apt-get install -y \
      python3-full \
      python3-dev \
      python3-venv \
      git \
      curl \
      ca-certificates \
      build-essential \
      postgresql-client \
      libpq-dev \
    && apt-get autoclean \
    && rm -rf /var/lib/{apt,dpkg,cache,log}

# Accept a build argument to optionally include the corporate certificate.
ARG INCLUDE_CORP_CERT=false
# If INCLUDE_CORP_CERT is true, copy the certificate from the build context,
# update the CA certificates and configure Git to use it. 
RUN if [ "$INCLUDE_CORP_CERT" = "true" ]; then \
      echo "Including corporate certificate..."; \
      if [ -f certs/corporate-ca.crt ]; then \
        cp certs/corporate-ca.crt /usr/local/share/ca-certificates/ && \
        update-ca-certificates && \
        git config --global http.sslCAInfo /usr/local/share/ca-certificates/corporate-ca.crt; \
      else \
        echo "Corporate certificate not found. Skipping..."; \
      fi; \
    fi

# (For development only: disable SSL verification for Git; remove in production)
ENV GIT_SSL_NO_VERIFY true

# Create a Python virtual environment and upgrade pip
RUN bash -c "python3 -m venv /home/venv && source /home/venv/bin/activate && pip install --upgrade pip"
ENV PATH="/home/venv/bin:$PATH"

# Configure pip to trust specific hosts
RUN pip config --global set global.trusted-host "pypi.org files.pythonhosted.org"

# Copy the requirements file for Layer 1 (ODC-related)
COPY docker/odc/requirements.odc.txt /tmp/requirements.odc.txt
RUN pip install --no-cache-dir -r /tmp/requirements.odc.txt

# Copy the requirements file for Layer 2 (other libraries)
COPY docker/odc/requirements.common.txt /tmp/requirements.common.txt
RUN pip install --no-cache-dir -r /tmp/requirements.common.txt

# Copy the configuration file
COPY ../datacube.conf /root/.datacube.conf

# Copy the entrypoint script into the image
COPY docker/odc/entrypoint.sh /home/app/entrypoint.sh

# Set working directory
WORKDIR /home/app

# Set the entrypoint so that it runs before the CMD
ENTRYPOINT ["/home/app/entrypoint.sh"]

# Set the default command
CMD ["stac-to-dc"]
