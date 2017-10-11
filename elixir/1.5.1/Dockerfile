#
# Based on Alpine Linux 3.6, with Erlang 20.1 [erts-9.1], Elixir 1.5.1 and Node.js 8.6.0
# Docker Hub repository: https://hub.docker.com/r/dreamingechoes/elixir
#
FROM alpine:3.6

# Install Erlang
ENV OTP_VERSION="20.1"

RUN set -xe \
    && OTP_DOWNLOAD_URL="https://github.com/erlang/otp/archive/OTP-${OTP_VERSION}.tar.gz" \
    && OTP_DOWNLOAD_SHA256="48dbd7bc884ce68ac97db26ce89716727ac63c7b2e0d46bc01fc527b9e428f64" \
    && apk add --no-cache --virtual .fetch-deps \
    	curl \
    	ca-certificates \
    && curl -fSL -o otp-src.tar.gz "$OTP_DOWNLOAD_URL" \
    && echo "$OTP_DOWNLOAD_SHA256  otp-src.tar.gz" | sha256sum -c - \
    && apk add --no-cache --virtual .build-deps \
    	dpkg-dev dpkg \
    	gcc \
    	g++ \
    	libc-dev \
    	linux-headers \
    	make \
    	autoconf \
    	ncurses-dev \
    	openssl-dev \
    	unixodbc-dev \
    	lksctp-tools-dev \
    	tar \
    && export ERL_TOP="/usr/src/otp_src_${OTP_VERSION%%@*}" \
    && mkdir -vp $ERL_TOP \
    && tar -xzf otp-src.tar.gz -C $ERL_TOP --strip-components=1 \
    && rm otp-src.tar.gz \
    && ( cd $ERL_TOP \
      && ./otp_build autoconf \
      && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
      && ./configure --build="$gnuArch" \
      && make -j$(getconf _NPROCESSORS_ONLN) \
      && make install ) \
    && rm -rf $ERL_TOP \
    && find /usr/local -regex '/usr/local/lib/erlang/\(lib/\|erts-\).*/\(man\|doc\|obj\|c_src\|emacs\|info\|examples\)' | xargs rm -rf \
    && find /usr/local -name src | xargs -r find | grep -v '\.hrl$' | xargs rm -v || true \
    && find /usr/local -name src | xargs -r find | xargs rmdir -vp || true \
    && scanelf --nobanner -E ET_EXEC -BF '%F' --recursive /usr/local | xargs -r strip --strip-all \
    && scanelf --nobanner -E ET_DYN -BF '%F' --recursive /usr/local | xargs -r strip --strip-unneeded \
    && runDeps="$( \
    	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
    		| tr ',' '\n' \
    		| sort -u \
    		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
    && apk add --virtual .erlang-rundeps $runDeps lksctp-tools \
    && apk del .fetch-deps .build-deps

# Install Elixir
ENV ELIXIR_VERSION="v1.5.1" \
    LANG=C.UTF-8

RUN set -xe \
    && ELIXIR_DOWNLOAD_URL="https://github.com/elixir-lang/elixir/releases/download/${ELIXIR_VERSION}/Precompiled.zip" \
    && ELIXIR_DOWNLOAD_SHA256="84af6eb4cb68d0f60b3edf4e275eb024f8eb8cccae91b18c2bbbc4b70a88934f" \
    && buildDeps=' \
    	ca-certificates \
    	curl \
    	unzip \
    ' \
    && apk add --no-cache --virtual .build-deps $buildDeps \
    && curl -fSL -o elixir-precompiled.zip $ELIXIR_DOWNLOAD_URL \
    && echo "$ELIXIR_DOWNLOAD_SHA256  elixir-precompiled.zip" | sha256sum -c - \
    && unzip -d /usr/local elixir-precompiled.zip \
    && rm elixir-precompiled.zip \
    && apk del .build-deps

# Install node and npm5
ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 8.6.0

RUN addgroup -g 1000 node \
    && adduser -u 1000 -G node -s /bin/sh -D node \
    && apk add --no-cache \
        libstdc++ \
    && apk add --no-cache --virtual .build-deps \
        binutils-gold \
        curl \
        g++ \
        gcc \
        gnupg \
        libgcc \
        linux-headers \
        make \
        python \
    # gpg keys listed at https://github.com/nodejs/node#release-team
    && for key in \
      9554F04D7259F04124DE6B476D5A82AC7E37093B \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      FD3A5288F042B6850C66B31F09FE44734EB7990E \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      B9AE9905FFD7803F25714661B63B535A4C206CA9 \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      56730D5401028683275BD23C23EFEFE93C4CFFFE \
    ; do \
      gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
      gpg --keyserver keyserver.pgp.com --recv-keys "$key" || \
      gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" ; \
    done \
      && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
      && curl -SLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
      && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
      && grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
      && tar -xf "node-v$NODE_VERSION.tar.xz" \
      && cd "node-v$NODE_VERSION" \
      && ./configure \
      && make -j$(getconf _NPROCESSORS_ONLN) \
      && make install \
      && apk del .build-deps \
      && cd .. \
      && rm -Rf "node-v$NODE_VERSION" \
      && rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt

ENV YARN_VERSION 1.1.0

RUN apk add --no-cache --virtual .build-deps-yarn curl gnupg tar \
    && for key in \
      6A010C5166006599AA17F08146C2130DFD2497F5 \
    ; do \
      gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
      gpg --keyserver keyserver.pgp.com --recv-keys "$key" || \
      gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" ; \
    done \
    && curl -fSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" \
    && curl -fSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" \
    && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
    && mkdir -p /opt/yarn \
    && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/yarn --strip-components=1 \
    && ln -s /opt/yarn/bin/yarn /usr/local/bin/yarn \
    && ln -s /opt/yarn/bin/yarn /usr/local/bin/yarnpkg \
    && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz \
    && apk del .build-deps-yarn

RUN apk update && \
    apk upgrade && \
    apk add git g++ gcc libgcc libstdc++ linux-headers make python && \
    apk update && \
    npm install -g npm5
