POP3 → Gmail Importer
======================

This folder contains deployment metadata and containerization for the upstream project: https://github.com/tejastice/pop3-gmail-importer

Quick overview
- Add the upstream repository as a git submodule inside this folder (note: we keep the upstream code in `pop3-gmail-importer-upstream` to separate it from the deployment):

  ```bash
  git submodule add https://github.com/tejastice/pop3-gmail-importer.git Pop3Sync/pop3-gmail-importer-upstream
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
4. Start services (on the device):

  ```bash
  cd Pop3Sync/pop3-gmail-importer
  docker compose up -d
  ```

Headless token generation & migration
------------------------------------
If your target device is headless, perform the OAuth flow on another machine (e.g., your laptop), then copy the resulting token files to the device. This avoids requiring an interactive browser on the headless device.

Steps:

1. On a machine with a browser, either clone the upstream repo or use the included submodule snapshot and prepare the environment. If you are working inside this repository, use the submodule path:

```bash
cd Pop3Sync/pop3-gmail-importer-upstream
python -m venv venv
source venv/bin/activate         # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env             # edit values to point to local files
```

Or, clone directly to a separate machine and follow the same steps.
2. Place your `credentials.json` in the project root (the script expects `credentials.json` path as configured in `.env`).

3. Run the connection test to perform OAuth and create token files:

```bash
python test_connection.py
```

When the OAuth flow runs, a browser will open and prompt you to allow access. The script saves tokens under the path specified in `.env` (e.g., `tokens/token_account1.json`).

4. Securely copy the token files to the headless device and place them under the persistent mount:

```bash
# On laptop (example):
scp tokens/token_account1.json user@device:/mnt/sdcard/pop3-gmail-importer/tokens/
# On the device, set strict permissions:
ssh user@device 'chmod 600 /mnt/sdcard/pop3-gmail-importer/tokens/token_account1.json'
```

5. Verify on the device that the container can load and refresh the token by running the test-connection logic (non-interactive):

```bash
# This will check POP3 and attempt to use the token; it will not open a browser
docker compose run --rm pop3-gmail-importer python test_connection.py
```

Notes & best practices
- Refresh tokens issued to a **published** (non-Testing) Google OAuth app are long-lived. Test-mode tokens expire after 7 days—publish your OAuth consent screen to production to avoid that.
- Always set token files to `600` and keep them on the device under `/mnt/sdcard/pop3-gmail-importer/tokens/`.
- If a token is revoked or fails to refresh, re-run the OAuth flow on a machine with a browser and repeat the copy step.
- Keep `credentials.json`, tokens, `.env`, `state/`, `backup/`, and `logs/` out of git and backed up securely.

Security notes
- Files `credentials.json`, `tokens/`, `.env`, `state/`, `backup/`, and `logs/` are all ignored by git and must never be checked in. Keep them on the device in `/mnt/sdcard/pop3-gmail-importer`.

