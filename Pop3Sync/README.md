POP3 → Gmail Importer
======================

This folder contains deployment metadata and containerization for the upstream project: https://github.com/tejastice/pop3-gmail-importer

Quick overview
- Add the upstream repository as a git submodule inside this folder:

  ```bash
  git submodule add https://github.com/tejastice/pop3-gmail-importer.git Pop3Sync/pop3-gmail-importer
  git submodule update --init --recursive
  ```

- Copy `.env.example` to `.env` in this folder and edit account settings.
- Place your `credentials.json` and any token files on the device at `/mnt/sdcard/pop3-gmail-importer` or inside the `tokens/` subfolder as described below.

Where to put credentials and environment files
- `.env` (container env_file): Place at `Pop3Sync/.env` (copy from `.env.example`). The compose file uses this `.env` to pass configuration into the container.
- `credentials.json` (Google OAuth client secrets): Put at `/mnt/sdcard/pop3-gmail-importer/credentials.json` on the device. Inside the container this will be available at `/data/credentials.json`.
- OAuth token files: Put under `/mnt/sdcard/pop3-gmail-importer/tokens/` (e.g., `/mnt/sdcard/pop3-gmail-importer/tokens/token_account1.json`). Example env var: `ACCOUNT1_GMAIL_TOKEN_FILE=/data/tokens/token_account1.json`.
- Other runtime files that must persist: `state/`, `backup/`, `logs/` — these are stored under `/mnt/sdcard/pop3-gmail-importer`.

Testing & first run
1. Ensure the `pop3-gmail-importer` submodule is present (see step above).
2. Copy `.env.example` to `.env` and update values (notably `ACCOUNT*_GMAIL_CREDENTIALS_FILE` and `ACCOUNT*_GMAIL_TOKEN_FILE`).
3. Place `credentials.json` into `/mnt/sdcard/pop3-gmail-importer/credentials.json` and create directory `/mnt/sdcard/pop3-gmail-importer/tokens/` with proper permissions.
4. Start services:

  ```bash
  cd Pop3Sync
  docker compose up -d
  ```

5. For an initial OAuth login you can run the `test-connection` service (it will open a browser on first run so you can authenticate):

  ```bash
  docker compose run --rm test-connection
  ```

Security notes
- Files `credentials.json`, `tokens/`, `.env`, `state/`, `backup/`, and `logs/` are all ignored by git and must never be checked in. Keep them on the device in `/mnt/sdcard/pop3-gmail-importer`.

