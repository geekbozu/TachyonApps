FROM python:3.11-slim

# Use an unprivileged user for runtime
RUN useradd -m -u 1000 runner || true

WORKDIR /app

# Copy upstream source (submodule) into image; assumes submodule is present at build time
COPY pop3-gmail-importer/ /app/

RUN pip install --no-cache-dir -r requirements.txt

ENV PYTHONUNBUFFERED=1

# Runtime data directory (mounted at runtime)
VOLUME ["/data"]

# Create data directories with safe permissions
RUN mkdir -p /data/tokens /data/state /data/backup /data/logs && chown -R runner:runner /data && chmod 700 /data/tokens

USER runner

CMD ["python", "main.py"]
