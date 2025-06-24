# The different stages of this Dockerfile are meant to be built into separate images
# https://docs.docker.com/develop/develop-images/multistage-build/#stop-at-a-specific-build-stage
# https://docs.docker.com/compose/compose-file/#target

FROM dunglas/frankenphp:builder AS builder

# Copy xcaddy in the builder image
COPY --from=caddy:builder /usr/bin/xcaddy /usr/bin/xcaddy

# CGO must be enabled to build FrankenPHP
RUN CGO_ENABLED=1 \
    XCADDY_SETCAP=1 \
    XCADDY_GO_BUILD_FLAGS="-ldflags='-w -s' -tags=nobadger,nomysql,nopgx" \
    CGO_CFLAGS=$(php-config --includes) \
    CGO_LDFLAGS="$(php-config --ldflags) $(php-config --libs)" \
    xcaddy build \
        --output /usr/local/bin/frankenphp \
        --with github.com/dunglas/frankenphp=./ \
        --with github.com/dunglas/frankenphp/caddy=./caddy/ \
        --with github.com/dunglas/caddy-cbrotli \
        # Mercure and Vulcain are included in the official build, but feel free to remove them
        --with github.com/dunglas/mercure/caddy \
        --with github.com/dunglas/vulcain/caddy
        # Add extra Caddy modules here

FROM dunglas/frankenphp AS frankenphp_base

# Replace the official binary by the one contained your custom modules
COPY --from=builder /usr/local/bin/frankenphp /usr/local/bin/frankenphp

# persistent / runtime deps
# hadolint ignore=DL3008
RUN apt-get update && apt-get install -y --no-install-recommends \
	acl \
	file \
	gettext \
	git \
	curl \
	bash \
	&& rm -rf /var/lib/apt/lists/*

# Install Node.js 22 (LTS) for frontend development
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get update \
    && apt-get install -y nodejs \
    && node --version && npm --version

RUN set -eux; \
    install-php-extensions \
    @composer \
	pdo_mysql \
	apcu \
	gd \
	intl \
	zip \
	opcache \
    xdebug \
    pcntl \
;

RUN composer global require laravel/installer;
ENV PATH="$PATH:/config/composer/vendor/bin"

# Set bash as default shell
SHELL ["/bin/bash", "-c"]

# Dev FrankenPHP images
FROM frankenphp_base AS frankenphp_dev

ENV APP_ENV=dev
ENV XDEBUG_MODE=off
ENV FRANKENPHP_WORKER_CONFIG=watch

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

CMD [ "frankenphp", "run", "--config", "/etc/frankenphp/Caddyfile", "--watch" ]

# Production FrankenPHP image
FROM frankenphp_base AS frankenphp_prod

ENV APP_ENV=prod
ENV XDEBUG_MODE=off

# Use production PHP configuration
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# No dev extensions installed in production

CMD [ "frankenphp", "run", "--config", "/etc/frankenphp/Caddyfile" ]
