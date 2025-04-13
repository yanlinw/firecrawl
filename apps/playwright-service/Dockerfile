FROM --platform=linux/amd64 mcr.microsoft.com/playwright:v1.41.1-jammy

# Install Python3 first
RUN set -x \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* /var/cache/apt/*.bin /var/log/* /tmp/* /var/tmp/* \
    && echo "APT::Get::AllowUnauthenticated \"true\";" > /etc/apt/apt.conf.d/99allow-unauth \
    && apt-get clean \
    && apt-get update -o Acquire::AllowInsecureRepositories=true \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends --allow-unauthenticated python3 \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* /var/cache/apt/*.bin /var/log/* /tmp/* /var/tmp/*

# Install pip separately
RUN set -x \
    && apt-get update -o Acquire::AllowInsecureRepositories=true \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends --allow-unauthenticated python3-pip \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* /var/cache/apt/*.bin \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && ln -sf /usr/bin/pip3 /usr/bin/pip

# Install libstdc++6 separately
RUN set -x \
    && apt-get update -o Acquire::AllowInsecureRepositories=true \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends --allow-unauthenticated libstdc++6 \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* /var/cache/apt/*.bin

# Set working directory and env vars
WORKDIR /app
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PIP_DISABLE_PIP_VERSION_CHECK=1

# Install Python dependencies
COPY requirements.txt ./

# Remove py which is pulled in by retry, py is not needed and is a CVE
RUN pip install --no-cache-dir --upgrade -r requirements.txt && \
    pip uninstall -y py && \
    playwright install chromium && playwright install-deps chromium && \
    ln -s /usr/local/bin/supervisord /usr/bin/supervisord

# Cleanup for CVEs and size reduction
# https://github.com/tornadoweb/tornado/issues/3107
# xserver-common and xvfb included by playwright installation but not needed after
# perl-base is part of the base Python Debian image but not needed for Danswer functionality
# perl-base could only be removed with --allow-remove-essential

COPY . ./

EXPOSE $PORT
# run fast api hypercorn
CMD hypercorn main:app --bind [::]:$PORT
# CMD ["hypercorn", "main:app", "--bind", "[::]:$PORT"]
# CMD ["sh", "-c", "uvicorn main:app --host 0.0.0.0 --port $PORT"]
