ARG BUILD_FROM
FROM $BUILD_FROM

# Dependencies - matches paperless-ng:master Dockerfile
RUN apt-get update \
    && apt-get -y --no-install-recommends install \
    curl \
    file \
    # fonts for test file thumbnail generation
    fonts-liberation \
    # for making translations further down
    gettext \
    gnupg \
    imagemagick \
    # for Numpy
    libatlas-base-dev \
    libxslt1-dev \
    mime-support \
    optipng \
    sudo \
    tzdata \
    # OCRmyPDF dependencies
    ghostscript \
    icc-profiles-free \
    liblept5 \
    libxml2 \
    pngquant \
    qpdf \
    tesseract-ocr \
    tesseract-ocr-eng \
    tesseract-ocr-deu \
    tesseract-ocr-fra \
    tesseract-ocr-ita \
    tesseract-ocr-spa \
    unpaper \
    zlib1g \
    && mkdir /var/log/supervisord /var/run/supervisord

# Download release of Paperless-ng, adding and removing wget/7zip
RUN apt-get update && apt-get install --no-install-recommends -y wget p7zip-full \
    && wget -O paperless.tar.xz https://github.com/jonaswinkler/paperless-ng/releases/download/ng-1.1.4/paperless-ng-1.1.4.tar.xz \
    && 7z x paperless.tar.xz \
    && 7z x paperless.tar -o/usr/src/ \
    && rm paperless.tar.xz paperless.tar \
    && cd /usr/src \
    && apt-get -y remove --purge wget p7zip-full \
    && rm -rf /var/lib/apt/lists/* \
    # Copy the app to the right place
    && mkdir -p /usr/src/paperless/src \
    && cp /usr/src/paperless-ng/requirements.txt /usr/src/paperless/requirements.txt \
    # copy app
    && cd /usr/src/paperless-ng \
    && cp -R ./src /usr/src/paperless/

# Install requirements and any temporary tools required 
RUN apt-get update && apt-get -y --no-install-recommends install \
    build-essential \
    libpoppler-cpp-dev \
    libpq-dev \
    libqpdf-dev \
    python3-dev \
    python3-pip \
    && python3 -m pip install --upgrade --no-cache-dir pip supervisor setuptools \
    && cd /usr/src/paperless \
    && python3 -m pip install --no-cache-dir -r requirements.txt \
    && apt-get -y purge build-essential libqpdf-dev python3-dev python3-pip \
    && apt-get -y autoremove --purge \
    && rm -rf /var/lib/apt/lists/*

#### Redis Setup
RUN apt-get update && \
    apt-get install --no-install-recommends python3 redis-server redis-tools -y\
    && sed -i "s/bind .*/bind 127.0.0.1/g" /etc/redis/redis.conf \
    && rm -rf /var/lib/apt/lists/*

# Clean Up
#RUN apt-get -y purge build-essential libqpdf-dev \
#    && apt-get -y autoremove --purge \
#    && rm -rf /var/lib/apt/lists/* 

# copy scripts
# FROM ORIGINAL DOCKER FILE: this fixes issues with imagemagick and PDF
RUN cd /usr/src/paperless-ng && \
    cp docker/imagemagick-policy.xml /etc/ImageMagick-6/policy.xml \
    && cp gunicorn.conf.py /usr/src/paperless \
    && cp docker/supervisord.conf /etc/supervisord.conf
# RUN cp docker/docker-entrypoint.sh /sbin/docker-entrypoint.sh

COPY scripts/docker-entrypoint.sh /sbin/docker-entrypoint.sh

WORKDIR /usr/src/paperless/src/

# add users, setup scripts
RUN addgroup --gid 1000 paperless \
    && useradd --uid 1000 --gid paperless --home-dir /usr/src/paperless paperless \
    && chown -R paperless:paperless ../ \
    && chmod 755 /sbin/docker-entrypoint.sh


ENV PAPERLESS_REDIS="redis://localhost:6379"
COPY scripts/wait-for-redis.sh scripts/wait-for-redis.sh

COPY scripts/setup_superuser.py scripts/setup_superuser.py

# Prepare the application
RUN sudo chmod +x scripts/wait-for-redis.sh \
    && sudo -HEu paperless python3 manage.py collectstatic --clear --no-input \
    && sudo -HEu paperless python3 manage.py compilemessages

VOLUME ["/usr/src/paperless/data", "/usr/src/paperless/media", "/usr/src/paperless/consume", "/usr/src/paperless/export"]

ENTRYPOINT ["/bin/bash", "/sbin/docker-entrypoint.sh"]
EXPOSE 8000

CMD ["/usr/local/bin/supervisord", "-c", "/etc/supervisord.conf"]

LABEL maintainer="Moshe <34072688+TheBestMoshe@users.noreply.github.com>"
