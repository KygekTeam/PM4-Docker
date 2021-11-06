# Build stage for compilation. Build tools like g++ will not be copied into the final stage to reduce image size.
FROM ubuntu:18.04

RUN apt-get update && apt-get install --no-install-recommends -y curl ca-certificates build-essential m4 gzip bzip2 bison git cmake autoconf automake pkg-config libtool libtool-bin re2c

RUN mkdir /build
WORKDIR /build
RUN git clone https://github.com/pmmp/PocketMine-MP.git -b master --recursive .
WORKDIR /build/build/php
RUN ./compile.sh -t linux64 -j ${THREADS:-$(grep -E ^processor /proc/cpuinfo | wc -l)} -f -g
RUN ln -s /build/build/php/bin/php7/bin/php /usr/bin/php

WORKDIR /build
RUN curl -L https://getcomposer.org/installer | php
RUN mv composer.phar /usr/bin/composer

# Set enable-dev-builds to true in pocketmine.yml to prevent startup errors
RUN php -r "file_put_contents('resources/pocketmine.yml', str_replace('enable-dev-builds: false', 'enable-dev-builds: true', file_get_contents('resources/pocketmine.yml')));"
RUN composer install --no-dev --prefer-dist --no-interaction --ignore-platform-reqs
RUN php vendor/pocketmine/bedrock-data/.minify_json.php
RUN php -dphar.readonly=0 build/server-phar.php
# Just to make sure DevTools didn't false-positive-exit
RUN test -f /build/PocketMine-MP.phar

FROM ubuntu:18.04
LABEL maintainer="KygekTeam <kygekteam@hotmail.com>"

RUN apt-get update && apt-get install --no-install-recommends -y ca-certificates wget

RUN groupadd -g 1000 pocketmine
RUN useradd -r -d /pocketmine -p "" -u 1000 -m -g pocketmine pocketmine

WORKDIR /pocketmine
COPY --from=0 /build/build/php/bin/php7 /usr/php
RUN grep -q '^extension_dir' /usr/php/bin/php.ini && \
	sed -ibak "s{^extension_dir=.*{extension_dir=\"$(find /usr/php -name *debug-zts*)\"{" /usr/php/bin/php.ini || echo "extension_dir=\"$(find /usr/php -name *debug-zts*)\"" >> /usr/php/bin/php.ini
RUN ln -s /usr/php/bin/php /usr/bin/php
COPY --from=0 /build/PocketMine-MP.phar PocketMine-MP.phar
ADD start.sh /usr/bin/start-pocketmine

RUN mkdir /plugins /data
RUN chown 1000:1000 /plugins /data . -R
RUN chmod o+x /usr/bin/php

USER pocketmine

ENV TERM=xterm

EXPOSE 19132/tcp
EXPOSE 19132/udp

VOLUME ["/data", "/plugins"]

CMD ["start-pocketmine"]
