# base defines a base stage that uses the official python runtime base image
FROM python:3.11.9-slim-bookworm AS base

# Add curl for healthcheck and libcap2-bin for privileged port binding
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl libcap2-bin && \
    setcap 'cap_net_bind_service=+ep' $(readlink -f $(which python3)) && \
    rm -rf /var/lib/apt/lists/*

# Create dedicated non-root user and group (UID/GID 10001)
RUN groupadd -g 10001 appgroup && \
    useradd -u 10001 -g appgroup -s /bin/sh -d /usr/local/app appuser

# Set the application directory
WORKDIR /usr/local/app

# Install our requirements.txt
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Ensure application directory ownership
RUN chown -R 10001:10001 /usr/local/app

# dev defines a stage for development, where it'll watch for filesystem changes
FROM base AS dev
RUN pip install --no-cache-dir watchdog
ENV FLASK_ENV=development
USER 10001
CMD ["python", "app.py"]

# final defines the stage that will bundle the application for production
FROM base AS final

# Copy our code from the current folder to the working directory inside the container
COPY --chown=10001:10001 . .

# Make port 80 available for links and/or publish
EXPOSE 80

USER 10001

# Define our command to be run when launching the container
CMD ["gunicorn", "app:app", "-b", "0.0.0.0:80", "--log-file", "-", "--access-logfile", "-", "--workers", "4", "--keep-alive", "0"]
