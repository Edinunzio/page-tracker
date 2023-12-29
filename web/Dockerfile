FROM python:3.11.2-slim-bullseye AS builder

# patches most recent security & bug fixes since image last published
RUN apt-get update && \
    apt-get upgrade --yes

# create and switch to regular user without admin privileges
RUN useradd --create-home realpython
USER realpython
WORKDIR /home/realpython

# create and activate virtual env

# define helper variable, VIRTUALENV
ENV VIRTUALENV=/home/realpython/venv
RUN python3 -m venv $VIRTUALENV

# update PATH var instead of activating new env
ENV PATH="$VIRTUALENV/bin:$PATH"

# copy project metadata from host machine to Docker image:
# chown to regular user!
COPY --chown=realpython pyproject.toml constraints.txt ./

# upgrade pip and setuptools before installing dependencies!!
# then install 3rd party libs
RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir -c constraints.txt ".[dev]"

# copy source code and run tests with linters and other static analysis tools
COPY --chown=realpython src/ src/
COPY --chown=realpython test/ test/

# needed for continuous integration. if any below returns non zero status, build will fail
RUN python -m pip install . -c constraints.txt && \
    python -m pytest test/unit/ && \
    python -m flake8 src/ && \
    python -m isort src/ --check && \
    python -m black src/ --check --quiet && \
    python -m pylint src/ --disable=C0114,C0116,R1705 && \
    python -m pip wheel --wheel-dir dist/ . -c constraints.txt
    # python -m bandit -r src/ --quiet

FROM python:3.11.2-slim-bullseye

RUN apt-get update && \
    apt-get upgrade --yes

RUN useradd --create-home realpython
USER realpython
WORKDIR /home/realpython

ENV VIRTUALENV=/home/realpython/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --from=builder /home/realpython/dist/page_tracker*.whl /home/realpython

RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir page_tracker*.whl

CMD ["flask", "--app", "page_tracker.app", "run", \
     "--host", "0.0.0.0", "--port", "5000"]
