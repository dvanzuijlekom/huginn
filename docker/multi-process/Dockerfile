FROM ubuntu:18.04

COPY docker/scripts/prepare /scripts/
RUN /scripts/prepare

COPY docker/multi-process/scripts/standalone-packages /scripts/
RUN /scripts/standalone-packages

WORKDIR /app

ENV HOME=/app

ARG UID=1001
RUN useradd -u "$UID" -g 0 -d /app -s /sbin/nologin -c "default user" default

ENV LC_ALL=en_US.UTF-8
ENV RAILS_ENV=production

COPY ["Gemfile", "Gemfile.lock", "/app/"]
COPY lib/gemfile_helper.rb /app/lib/
COPY vendor/gems/ /app/vendor/gems/

# Get rid of annoying "fatal: Not a git repository (or any of the parent directories): .git" messages
RUN umask 002 && git init && \
    bundle config set --local path vendor/bundle && \
    bundle config set --local without 'test development' && \
    APP_SECRET_TOKEN=secret DATABASE_ADAPTER=mysql2 ON_HEROKU=true bundle install -j 4

COPY ./ /app/

ARG OUTDATED_DOCKER_IMAGE_NAMESPACE=false
ENV OUTDATED_DOCKER_IMAGE_NAMESPACE=${OUTDATED_DOCKER_IMAGE_NAMESPACE}

RUN umask 002 && \
    APP_SECRET_TOKEN=secret DATABASE_ADAPTER=mysql2 ON_HEROKU=true bundle exec rake assets:clean assets:precompile && \
    chmod g=u /app/.env.example /app/Gemfile.lock /app/config/ /app/tmp/ && \
    chown -R "$UID" /app

EXPOSE 3000

COPY docker/multi-process/scripts/supervisord.conf /etc/supervisor/
COPY ["docker/multi-process/scripts/bootstrap.conf", \
      "docker/multi-process/scripts/foreman.conf", \
      "docker/multi-process/scripts/mysqld.conf", "/etc/supervisor/conf.d/"]
COPY ["docker/multi-process/scripts/bootstrap.sh", \
      "docker/multi-process/scripts/foreman.sh", \
      "docker/multi-process/scripts/init", \
      "docker/scripts/setup_env", "/scripts/"]
CMD ["/scripts/init"]

USER $UID

VOLUME /var/lib/mysql
