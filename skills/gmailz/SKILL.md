---
name: gmailz
description: Multi-account Gmail Manager with search, sync, and caching capabilities.
---
# gmailz Package Documentation

## Executive Summary

**gmailz** is a comprehensive multi-account Gmail management package that orchestrates Gmail operations through both programmatic and CLI interfaces. It supports concurrent execution across dozens of accounts while implementing global rate limiting to ensure compliance with Gmail API quotas.

### Key Features
- **Multi-account management**: Handle multiple Gmail accounts simultaneously through a unified interface
- **Automated OAuth2 flow**: Browser-based authentication with credential capture and token management
- **Global rate limiting**: Built-in API call throttling to comply with Gmail API quotas
- **Message operations**: Search, fetch, send emails with HTML/attachments support
- **Label management**: Create, modify, and organize Gmail labels programmatically
- **Token persistence**: Secure credential and token storage using PostgreSQL backend
- **Connection testing**: Verify account connectivity and token validity
- **Excel file operations**: Read, analyze, and modify Excel (.xlsx) files using openpyxl

## Architecture Overview

```
gmailz/
├── src/
│   ├── __init__.py
│   ├── cli.py                    # CLI interface
│   ├── models.py                 # Pydantic data models
│   ├── runner.py                 # API operations runner
│   ├── constants.py              # Constants and enums
│   ├── logs.py                   # Logging configuration
│   ├── pass_dict.py              # Password/token utilities
│   ├── gmail_setup/              # Account setup workflows
│   │   ├── setup_pipeline.py     # Full setup orchestration
│   │   ├── oauth_handler.py      # OAuth2 token management
│   │   ├── oauth_navigation.py   # Browser navigation helpers
│   │   ├── credential_navigation.py  # Credential retrieval
│   │   ├── google_login.py       # Google login automation
│   │   └── oauth_catcher.py      # OAuth code capture
│   └── gmail_client/             # Core Gmail API client
│       ├── client.py             # GmailClient class
│       ├── parsing.py            # Message parsing utilities
│       ├── attachments.py        # Attachment handling
│       └── utils.py              # Text processing utilities
├── tests/                        # Test suite
└── gmail_config/                 # Configuration storage
    └── client_creds/            # OAuth credentials per account
```

## Quick Start

### CLI
```bash
# One-off setup for an account
python -m gmailz.gmail_setup.setup_pipeline user@example.com

# Verify connection (basic connectivity check; does not filter by sender)
python -m gmailz.cli user@example.com verify

# Fetch latest unread message from a specific sender (example: your brother Stijn)
python -m gmailz.cli user@example.com fetch --query "from:stijn" --limit 1
```

### Python (fetch latest from a sender and save attachments)
```python
import asyncio, json, os
from pathlib import Path
from gmailz.manager.account_manager import AccountManager
from sql_admin import quick_pool

async def main():
    pool = await quick_pool(server_name="postgres-server-mapping", db_name="gmail")
    mgr = AccountManager(pool)
    await mgr.load_accounts_from_config()
    account = mgr.accounts.get("twawouters@gmail.com")
    if not account:
        print("Account not loaded")
        return

    # get latest message from Stijn
    msgs = account.get_messages(query="from:stijn", max_results=1, user_id="me")
    if not msgs:
        print("No messages from Stijn")
        return

    msg = msgs[0]
    out = Path("latest_from_stijn")
    out.mkdir(exist_ok=True)

    # save metadata
    meta = {
        "id": msg.id,
        "sender": msg.sender,
        "subject": msg.subject,
        "date": str(msg.date),
        "snippet": msg.snippet,
        "headers": dict(msg.headers),
        "has_attachments": bool(msg.attachments),
    }
    (out / "message_meta.json").write_text(json.dumps(meta, indent=2, default=str))

    # save attachments
    for att in msg.attachments:
        data = account.get_attachment(att.attachment_id, msg.id, user_id="me")
        if data:
            name = att.filename or f"attach_{att.attachment_id[:8]}.bin"
            (out / name).write_bytes(data)
            print(f"Saved attachment: {name} ({len(data)} bytes)")

    print("Saved to:", os.path.abspath(str(out)))

asyncio.run(main())
```

### Excel File Operations
When working with Excel files (`.xlsx`), use the `excel-file-operations` skill:

```python
from excelz.read.core import read_excel_sheet_to_dict
from excelz.write.core import write_to_excel

# Read data from Excel
sheets = read_excel_sheet_to_dict("data.xlsx")
for sheet_idx, rows in enumerate(sheets):
    print(f"Sheet {sheet_idx}: {len(rows)} rows")
    for row in rows:
        print(row)

# Write data to Excel
write_to_excel(
    data=[["Name", "Amount"], ["Alice", 100], ["Bob", 200]],
    file_path="output.xlsx",
)
```

## API Reference

### Core Client: `GmailClient`

The primary entry point for programmatic Gmail operations. Initialize with credentials from your database:

```python
from gmailz.gmail_client.client import GmailClient

client = GmailClient(
    credentials_json=credentials_dict,
    token_json=token_dict
)

# Test connection
is_connected = client.test_connection()

# Search messages (query="" returns most recent)
messages = client.get_messages(query="", max_results=10)
for msg in messages:
    print(f"{msg.sender}: {msg.subject}")

# Send email
result = client.send_message(
    to="recipient@example.com",
    subject="Hello",
    body="Test message",
    attachments=["/path/to/file.pdf"]
)
```

#### Methods
- `test_connection(user_id="me")` → bool: Verify API connectivity
- `get_messages(query="", max_results=50, user_id="me")` → List[Message]: Search messages  
- `get_latest_message(user_id="me")` → Optional[Message]: Fetch most recent single message (use: `get_messages(query="", max_results=1)[0]`)
- `get_message(msg_id, user_id="me")` → Optional[Message]: Fetch single message
- `list_labels(user_id="me")` → List[Label]: Get all labels
- `get_label(label_id, user_id="me")` → Optional[Label]: Get specific label
- `create_label(name, user_id="me", ...)` → Optional[Label]: Create new label
- `delete_label(label_id, user_id="me")` → bool: Remove label
- `batch_modify_labels(message_ids, add_labels, remove_labels, user_id)` → int
- `send_message(to, subject, body, sender, ...)` → Optional[Dict]: Send email
- `modify_labels(msg_id, add_labels, remove_labels, user_id)` → bool
- `mark_as_read(msg_id, user_id)` → bool
- `mark_as_unread(msg_id, user_id)` → bool
- `archive(msg_id, user_id)` → bool
- `trash(msg_id, user_id)` → bool
- `star(msg_id, user_id)` → bool
- `unstar(msg_id, user_id)` → bool
- `download_attachments_for_message(message, download_dir, user_id)` → Dict
- `get_attachment(attachment_id, message_id, user_id)` → Optional[bytes]
- `inspect_labeled_messages(user_id, max_samples)` → None (debug)

### Data Models

#### `Account`
Represents a Gmail account configuration stored in the database:
- `email` (str): Account email address
- `credentials_json` (Optional[Dict]): OAuth client credentials
- `token_json` (Optional[Dict]): OAuth token data
- `created_at`, `updated_at` (Optional[datetime]): Timestamps
- `is_active` (bool): Account status
- `api_call_count` (int): Usage tracking

#### `Message`
Represents a Gmail message:
- `id`, `thread_id`, `sender`, `subject`, `date`
- `snippet`, `plain`, `html` (message content)
- `label_ids`, `headers`, `cc`, `bcc`
- `attachments` (List[Attachment])
- Properties: `labels`, `is_unread`, `is_starred`

#### `Label`
Represents a Gmail label:
- `id`, `name`, `type` ("user" or "system")
- `message_list_visibility`, `label_list_visibility`
- `unread_count`, `total_count`
- Properties: `is_system`, `is_user`

#### `SystemLabels` (class)
Constants for system label IDs:
- `INBOX`, `SPAM`, `TRASH`, `UNREAD`, `STARRED`
- `IMPORTANT`, `SENT`, `DRAFT`, `CHAT`

## Account Manager: `AccountManager`

Manages multiple accounts with database persistence:

```python
from gmailz.manager.account_manager import AccountManager
import asyncpg

pool = await asyncpg.create_pool(...)
manager = AccountManager(pool)

# Load all accounts from database
await manager.load_accounts_from_config()

# Run operation on all accounts concurrently
results = manager.run_on_all_accounts('get_messages', query="", max_results=1)

# Fetch a single "latest" message: get 1 most recent, then inspect
latest = manager.run_on_all_accounts('get_messages', query="", max_results=1)
for email, msgs in latest.items():
    if msgs:
        msg = msgs[0]  # most recent message
        print(f"{email}: {msg.subject}")
```

#### Key Methods
- `create(pool)` → AccountManager: Async factory method
- `load_accounts_from_config()` → None: Load all DB accounts
- `load_single_account_from_config(email, acc=None)` → Optional[Account]
- `run_on_all_accounts(method_name, *args, **kwargs)` → Dict[str, Any]
- `save_refreshed_token_to_db(email, account)` → bool

## OAuth2 Token Handling

The `oauth_handler.py` module manages the complete OAuth flow:

```python
from gmailz.gmail_setup.oauth_handler import (
    ensure_oauth2credentials_format,
    convert_to_oauth2credentials,
)

# Convert minimal token to OAuth2Credentials format
token = ensure_oauth2credentials_format(
    token_json,
    client_secret_json
)

# Or use the automated flow
from gmailz.gmail_setup.setup_pipeline import setup_new_account

success = await setup_new_account(
    email="user@example.com",
    password="password",
    manual_mode=False  # Use browser automation
)
```

### Token Format Conversion

The package converts standard Gmail API tokens to the `OAuth2Credentials` format required by `google-api-python-client`:

```python
# Token structure expected by gmailz:
{
    "access_token": "***",
    "client_id": "...",
    "client_secret": "...",
    "refresh_token": "***",
    "token_expiry": "2024-01-01T00:00:00+00:00",
    ...
}
```

## Global Rate Limiting

`AccountManager` implements automatic rate limiting with a minimum 50ms interval between API calls:

```python
# This is automatic - no configuration needed
# Each account operation respects the global interval
results = manager.run_on_all_accounts('get_messages')
```

## CLI Usage

```bash
# Interactive menu
python -m gmailz.cli

# Direct actions (CLI module)
python -m gmailz.cli user@example.com verify
python -m gmailz.cli user@example.com fetch

# GUI mode
python -m gmailz.cli -g
```

Note: direct CLI subcommands like `verify` and `fetch` are exposed via `gmailz.cli`.

## Setup Pipeline

Full account setup with browser automation:

```bash
# Single account setup
python -m gmailz.gmail_setup.setup_pipeline user@example.com

# With password (optional)
python -m gmailz.gmail_setup.setup_pipeline user@example.com mypassword
```

## Database Schema

Accounts are stored with the following fields:
- `email` (TEXT, PRIMARY KEY)
- `credentials_json` (JSONB)
- `token_json` (JSONB)
- `client_id_prefix` (TEXT)
- `created_at` (TIMESTAMP)
- `updated_at` (TIMESTAMP)
- `status` (TEXT) - active/incomplete/auth_error
- `is_active` (BOOLEAN)
- `api_call_count` (INTEGER)

## Common Pitfalls & Solutions

1. **Invalid Grant Error**: token expired or revoked → re-run `setup_pipeline`
2. **Access Denied**: insufficient OAuth consent → re-authorize via setup pipeline
3. **Invalid Client**: wrong credentials → re-download from Google Cloud Console
4. **Token Format Issues**: use `ensure_oauth2credentials_format()` for conversion
5. **Rate Limiting**: handled automatically by `AccountManager`
6. **Excel file confusion**: Remember — file-based operations use `excel-file-operations`, NOT live Excel

## Testing

```bash
pytest tests/ -xvs
```

## Dependencies

- `google-auth>=2.0.0`
- `google-auth-oauthlib>=0.5.0`
- `google-api-python-client>=2.0.0`
- `orjson>=3.6.0`
- `sqlalchemy` (via sql_admin)
- `asyncpg`
- `browserapi` (for automated browser flows)
- `openpyxl` (for Excel file operations)

## License

MIT