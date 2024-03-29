FROM ubuntu:18.04

COPY docker/scripts/prepare /scripts/
RUN /scripts/prepare

WORKDIR /app

COPY ["Gemfile", "Gemfile.lock", "/app/"]
COPY lib/gemfile_helper.rb /app/lib/
COPY vendor/gems/ /app/vendor/gems/

# Get rid of annoying "fatal: Not a git repository (or any of the parent directories): .git" messages
RUN umask 002 && git init && \
    LC_ALL=en_US.UTF-8 RAILS_ENV=production APP_SECRET_TOKEN=secret DATABASE_ADAPTER=mysql2 ON_HEROKU=true bundle install --without test development --path vendor/bundle -j 4

COPY ./ /app/

ARG OUTDATED_DOCKER_IMAGE_NAMESPACE=false
ENV OUTDATED_DOCKER_IMAGE_NAMESPACE ${OUTDATED_DOCKER_IMAGE_NAMESPACE}

RUN apt-get update && apt-get -y install libcap2-bin && \
    rm -rf /var/lib/apt/lists/* && \
    umask 002 && \
    bundle update mimemagic && \
    LC_ALL=en_US.UTF-8 RAILS_ENV=production APP_SECRET_TOKEN=secret DATABASE_ADAPTER=mysql2 ON_HEROKU=true bundle exec rake assets:clean assets:precompile && \
    chmod g=u /app/.env.example /app/Gemfile.lock /app/config/ /app/tmp/

EXPOSE 3000

COPY ["docker/scripts/setup_env", "docker/single-process/scripts/init", "/scripts/"]
RUN setcap CAP_NET_BIND_SERVICE=+eip /usr/bin/ruby2.5
CMD ["/scripts/init"]

USER 1001
