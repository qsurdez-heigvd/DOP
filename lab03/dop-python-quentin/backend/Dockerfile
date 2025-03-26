# The builder image, used to build the virtual environment
FROM python:3.13-alpine as builder

# Pin the poetry version so that no breaking changes can happen, only minor ones such as security updates
RUN pip install poetry==2.1

# Setting up variables to make sure the behavior of Poetry will be idempotent
ENV POETRY_NO_INTERACTION=1 \
 POETRY_VIRTUALENVS_IN_PROJECT=1 \
 POETRY_VIRTUALENVS_CREATE=1 \
 POETRY_CACHE_DIR=/tmp/poetry_cache

WORKDIR /app

COPY pyproject.toml poetry.lock ./

RUN touch README.md



RUN poetry install --no-root

# The runtime image, used to just run the code provided its virtual environment
# We use the apline version so that only the dependencies to run python are available
FROM python:3.13-alpine as runtime

# RUN apk add --no-cache \
#  musl-dev \
#  postgresql-dev

ENV VIRTUAL_ENV=/app/.venv \
 PATH="/app/.venv/bin:$PATH"

COPY --from=builder ${VIRTUAL_ENV} ${VIRTUAL_ENV}

COPY backend ./backend

EXPOSE 80

CMD python -m uvicorn backend.main:app --host 0.0.0.0 --port 80