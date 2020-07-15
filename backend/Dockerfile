FROM python:3.8-alpine AS base

# In theory, these environment variables should never need to be changed.
# Make sure that you have a good reason to override them.
ENV LISTEN_HOST 0.0.0.0
ENV LISTEN_PORT 80

ENV PYTHONUNBUFFERED 1

WORKDIR /backend

COPY backend/requirements.txt ./
RUN \
	apk update \
	&& apk add \
        --no-cache \
        --virtual build-dependencies \
        gcc \
        python3-dev \
        musl-dev \
        curl \
    && apk add \
        --no-cache \
        jpeg-dev \
        zlib-dev \
        freetype-dev \
        lcms2-dev \
        openjpeg-dev \
        tiff-dev \
        tk-dev \
        tcl-dev \
        harfbuzz-dev \
        fribidi-dev \
        postgresql-dev \
    && curl \
        -so /backend/wait-for.sh \
        https://raw.githubusercontent.com/eficode/wait-for/f71f8199a0dd95953752fb5d3f76f79ced16d47d/wait-for \
    && chmod +x /backend/wait-for.sh \
    && pip3 install -r requirements.txt \
    && apk del build-dependencies
COPY backend/myproject myproject
COPY backend/manage.py .
CMD ./manage.py collectstatic --noinput \
        && ./manage.py migrate \
        && gunicorn myproject.wsgi:application \
            --bind $LISTEN_HOST:$LISTEN_PORT \
            --log-level $GUNICORN_LOG_LEVEL \
            --workers $GUNICORN_WORKERS
ENV DJANGO_DATABASE_HOST database
ENV DJANGO_DATABASE_NAME postgres
ENV DJANGO_DATABASE_PORT 5432
ENV GUNICORN_LOG_LEVEL debug
ENV GUNICORN_WORKERS 1
CMD /backend/wait-for.sh \
    $DJANGO_DATABASE_HOST:$DJANGO_DATABASE_PORT \
        -- ./manage.py collectstatic --noinput \
        && ./manage.py migrate \
        && gunicorn myproject.wsgi:application \
            --bind $LISTEN_HOST:$LISTEN_PORT \
            --log-level $GUNICORN_LOG_LEVEL \
            --workers $GUNICORN_WORKERS \
            --reload