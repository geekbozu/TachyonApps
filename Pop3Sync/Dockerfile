FROM python:3.11-slim

# Use an unprivileged user for runtime
RUN useradd -m -u 1000 runner || true

WORKDIR /app

# Copy upstream source (submodule) into image; assumes submodule is present at build time
# The upstream submodule is stored at ../pop3-gmail-importer-upstream relative to the build context
COPY pop3-gmail-importer-upstream/ /app/

RUN pip install --no-cache-dir -r requirements.txt

ENV PYTHONUNBUFFERED=1

# Runtime data directory (mounted at runtime)
VOLUME ["/data"]

# Create data directories with safe permissions
RUN mkdir -p /data/tokens /data/state /data/backup /data/logs && chown -R runner:runner /data && chmod 700 /data/tokens

# Ensure /app/state exists and is owned by the unprivileged user so a named volume
# initialized from the image will have the correct ownership (avoids permission
# denied when the container runs as `runner`).
RUN mkdir -p /app/state && chown -R runner:runner /app/state && chmod 700 /app/state

USER runner

CMD ["python", "main.py"]
