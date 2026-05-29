---
name: m365-copilot
description: |
  Interact with Microsoft 365 Copilot (work version) via the browser  send
  prompts, read responses, start new chats, list chat history, and query work
  data. Use when the user wants to ask Copilot something, search work emails,
  find files, summarize meetings, use Microsoft Copilot at work, query their
  M365 data, or automate Copilot interactions. Activate on mentions of
  Copilot, M365 Copilot, Microsoft Copilot, work copilot, Copilot chat,
  "ask copilot", "copilot search", or Office 365 AI assistant.
allowed-tools: bash
---

# M365 Copilot

Control Microsoft 365 Copilot (work version) in the browser via UI automation and DOM reading. Requires the user to be logged into M365 Copilot at `m365.cloud.microsoft/chat`.

## Prerequisites

The user must be logged into M365 Copilot in the browser. The skill drives the chat UI directly  typing messages and reading responses from the DOM.

## Commands

```bash
m365-copilot ask <prompt>                 # Send a prompt and get the response
m365-copilot new                          # Start a new chat
m365-copilot history                      # List recent chat history from sidebar
m365-copilot read                         # Read all messages in current chat
m365-copilot mode <work|web>              # Switch between Work and Web mode
```

## Architecture

- **Chat interaction**: UI automation via playwright-cli (click, type, press Enter)
- **Response reading**: DOM queries on `[role=article]` elements
- **New chat**: Click "New chat" button
- **History**: Read sidebar chat list
- **Mode**: Click Work/Web toggle

The chat API itself (substrate.office.com/sydney) uses encrypted MSAL tokens in a SharedWorker, making direct API calls impractical. UI automation provides reliable access to all Copilot features including work data grounding, citations, and multi-turn conversations.

## Notes

- Responses may take 5-15 seconds depending on complexity
- Work mode queries emails, calendar, files, Teams messages
- Web mode uses Bing search
- The skill waits for Copilot to finish responding before returning
- Multi-turn conversations are supported (messages accumulate in the current chat)
