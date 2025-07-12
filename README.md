# 🔒 Calendar MCP - Minimal Permissions Fork

**A security-focused Google Calendar MCP server with minimal OAuth permissions**

> **Why this fork exists:** Giving an MCP server access to ALL your Google Workspace data (Gmail, Drive, Docs, Sheets, etc.) is very scary. This fork provides only the permissions you actually need for reading emails and read/write to calendars.

## 🚨 Security First

This is a **minimal permissions fork** of the original Google Workspace MCP server. Instead of requesting access to your entire Google account, this version only asks for:

### 🟢 Calendar-Only Mode (Default)

- ✅ `calendar.readonly` - Read your calendar events
- ✅ `calendar.events` - Create/modify calendar events
- ✅ `userinfo.email` + `openid` - Basic user identification
- ❌ **NO** access to Gmail, Drive, Docs, Sheets, or any other services

### 🟡 Calendar + Gmail Read Mode (Optional)

- ✅ Everything from Calendar-Only mode
- ✅ `gmail.readonly` - Read your emails (no sending/deleting)
- ❌ **NO** access to Drive, Docs, Sheets, or other services

**Compare this to the original:** The upstream version requests permissions for Gmail, Drive, Docs, Sheets, Slides, Forms, Tasks, Chat, and more - basically your entire Google account.

## 🚀 Installation

### For Friends/Family (Private Git Distribution)

```bash
# Install directly from this repository
uvx git+https://github.com/yourusername/google_workspace_mcp.git

# Or if you prefer calendar-only mode specifically
uvx git+https://github.com/yourusername/google_workspace_mcp.git calendar-mcp-minimal --scope-mode calendar-only
```

### Development/Local Use

```bash
git clone https://github.com/yourusername/google_workspace_mcp.git
cd google_workspace_mcp
uv run main.py --scope-mode calendar-only
```

## 🔧 Usage

### Calendar-Only Mode (Recommended)

```bash
calendar-mcp-minimal --scope-mode calendar-only --tools calendar
```

### Calendar + Gmail Read Mode

```bash
calendar-mcp-minimal --scope-mode calendar-gmail --tools calendar gmail
```

## ⚙️ Setup

### 1. Google Cloud Configuration

1. Create a Google Cloud Project
2. Enable **only** the APIs you need:
   - Google Calendar API (required)
   - Gmail API (optional, only if using calendar-gmail mode)
3. Create OAuth 2.0 credentials with redirect URI: `http://localhost:8000/oauth2callback`

### 2. Easy Installation Script

```bash
python install_claude.py
```

The installer will:

- Prompt you to choose between calendar-only or calendar+gmail modes
- Configure your OAuth credentials
- Set up Claude Desktop automatically

### 3. Manual Claude Desktop Config

Add to your Claude Desktop config:

```json
{
  "mcpServers": {
    "calendar-minimal": {
      "command": "uvx",
      "args": [
        "git+https://github.com/yourusername/google_workspace_mcp.git",
        "--scope-mode",
        "calendar-only"
      ],
      "env": {
        "GOOGLE_OAUTH_CLIENT_ID": "your-client-id",
        "GOOGLE_OAUTH_CLIENT_SECRET": "your-client-secret"
      }
    }
  }
}
```

## 🛡️ Security Benefits

| Original Workspace MCP                  | This Minimal Fork           |
| --------------------------------------- | --------------------------- |
| 📧 Full Gmail access (read/send/delete) | 🔒 Optional read-only Gmail |
| 📁 Full Google Drive access             | ❌ No Drive access          |
| 📄 Google Docs read/write               | ❌ No Docs access           |
| 📊 Google Sheets read/write             | ❌ No Sheets access         |
| 🖼️ Google Slides access                 | ❌ No Slides access         |
| 📝 Google Forms access                  | ❌ No Forms access          |
| ✓ Google Tasks access                   | ❌ No Tasks access          |
| 💬 Google Chat access                   | ❌ No Chat access           |
| **Total: 9+ Google services**           | **Total: 1-2 services max** |

## 🎯 Available Tools

### Calendar Tools

- `list_calendars` - List your calendars
- `get_events` - Get calendar events with filtering
- `create_event` - Create new events
- `modify_event` - Update existing events
- `delete_event` - Remove events

### Gmail Tools (Optional - only in calendar-gmail mode)

- `search_gmail_messages` - Search your emails (read-only)
- `get_gmail_message_content` - Read specific emails (read-only)

## 🔒 What This Fork Removes

**Deleted services for security:**

- `gdrive/` - Google Drive access removed
- `gdocs/` - Google Docs access removed
- `gsheets/` - Google Sheets access removed
- `gslides/` - Google Slides access removed
- `gforms/` - Google Forms access removed
- `gtasks/` - Google Tasks access removed
- `gchat/` - Google Chat access removed

**Result:** Minimal attack surface, minimal permissions, maximum security.

## 🤝 Contributing

This fork prioritizes security over features. When adding functionality:

1. Always use minimal required permissions
2. Default to read-only access when possible
3. Add scope modes rather than expanding default permissions
4. Document security implications of any new features

## 📄 License

MIT License - same as the original project.

## 🙏 Credits

Forked from [taylorwilsdon/google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp) - excellent original work, but we needed something more security-focused.

---

**Remember:** Only grant the minimum permissions your AI assistant actually needs. Your data security matters more than convenience features.

