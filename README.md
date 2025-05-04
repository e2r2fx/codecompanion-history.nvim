# CodeCompanion History Extension

[![Neovim](https://img.shields.io/badge/Neovim-57A143?style=flat-square&logo=neovim&logoColor=white)](https://neovim.io)
[![Lua](https://img.shields.io/badge/Lua-2C2D72?style=flat-square&logo=lua&logoColor=white)](https://www.lua.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](./CONTRIBUTING.md)

A history management extension for [codecompanion.nvim](https://codecompanion.olimorris.dev/) that enables saving, browsing and restoring chat sessions.

> [!CAUTION]
> **Warning**: This extension is not yet ready for use.

## Features

- 💾 Automatic chat session saving with context preservation
- 🎯 Smart title generation for chats 
- 🔄 Continue from where you left
- 📚 Browse saved chats with preview
- 🔍 Multiple picker interfaces
- ⚡ Restore chat sessions with full context and tools state

## Requirements

- Neovim >= 0.8.0
- [codecompanion.nvim](https://codecompanion.olimorris.dev/)
- [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) (optional, for enhanced picker)

## Installation

Using [lazy.nvim](https://github.com/folke/lazy.nvim):

### First install the plugin

```lua
{
    "olimorris/codecompanion.nvim",
    dependencies = {
        --other plugins
        "ravitemer/codecompanion-history.nvim"
    }
}
```

### Add history extension to CodeCompanion config

```lua
require("codecompanion").setup({
    extensions = {
        history = {
            enabled = true,
            opts = {
                -- Keymap to open history from chat buffer (default: gh)
                keymap = "gh",
                -- Automatically generate titles for new chats
                auto_generate_title = true,
                ---On exiting and entering neovim, loads the last chat on opening chat
                continue_last_chat = false,
                ---When chat is cleared with `gx` delete the chat from history
                delete_on_clearing_chat = false,
                -- Picker interface ("telescope" or "default")
                picker = "telescope",
                ---Enable detailed logging for history extension
                enable_logging = false,
            }
        }
    }
})
```

## Usage 

#### Commands

- `:CodeCompanionHistory` - Open the history browser

#### Chat Buffer Keymaps

- `gh` - Open history browser (customizable via `opts.keymap`)

#### History browser

The history browser shows all your saved chats with:
- Title (auto-generated or custom)
- Last updated time  
- Preview of chat contents

Actions in history browser:
- `<CR>` - Open selected chat
- `x` - Delete selected chat (Doesn't apply to default vim.ui.select)

#### Title Generation

Chat titles are automatically generated based on the context of your conversation. You can:

- Let the extension auto-generate titles (controlled by `auto_generate_title`)
- See the titles updated in real-time as you chat

## How It Works


```mermaid
graph TD
    subgraph CodeCompanion Core Lifecycle
        A[CodeCompanionChatCreated Event] --> B{Chat Submitted};
        B --> C[LLM Response Received];
        subgraph Chat End
            direction RL
            D[CodeCompanionChatCleared Event];
        end
        C --> D;
        B --> D;
    end

    subgraph Extension Integration
        A -- Extension Hooks --> E[Init & Subscribe];
        E --> F[Setup Auto-Save];
        F --> G[Prepare Auto-Title];

        C -- Extension Hooks --> H[Subscriber Triggered];
        H --> I[Auto-Save Chat State - Messages, Tools, Refs];
        I --> J{No Title & Auto-Title Enabled?};
        J -- Yes --> K[Generate Title];
        K --> L[Update Buffer Title];
        L --> M[Save Chat with New Title];
        J -- No --> B;
        M --> B;

        D -- Extension Hooks --> N[Respond to Clear Event];
        N --> O[Delete Chat from Storage];
        O --> P[Reset Extension State - Title/ID];
    end

    subgraph User History Interaction
        Q[User Action - gh / :CodeCompanionHistory] --> R{History Browser};
        R -- Restore --> S[Load Chat State from Storage];
        S --> A;
        R -- Delete --> O;
    end
```

Here's what's happening in simple terms:

1. When you create a new chat, our extension jumps in and sets up two things:
   - An autosave system that will save your chat
   - A title generator that will name your chat based on the conversation

2. As you chat:
   - When you submit a message, we listen to `CodeCompanionChatSubmitted` event and save state
   - Every time you get a response, our extension automatically saves everything 
   - If your chat doesn't have a title yet, it tries to create one that makes sense
   - All your messages, tools, and references are safely stored

3. When you clear a chat:
   - Our extension knows to remove it from storage (if configured)
   - This keeps your history clean and organized

4. Any time you want to look at old chats:
   - Use `gh` or the command to open the history browser
   - Pick any chat to restore it completely
   - Or remove ones you don't need anymore

<details>
    <summary> Technical details </summary>

The extension integrates with CodeCompanion through a robust event-driven architecture:

1. **Initialization and Storage Management**:
   - Uses a dedicated Storage class to manage chat persistence in `{data_path}/codecompanion-history/`
   - Maintains an index.json for metadata and individual JSON files for each chat
   - Implements file I/O operations with error handling and atomic writes

2. **Chat Lifecycle Integration**:
   - Hooks into `CodeCompanionChatCreated` event to:
     - Generate unique save_id (Unix timestamp)
     - Initialize chat subscribers for auto-saving
     - Set initial buffer title with sparkle icon (✨)
   
   - Monitors `CodeCompanionChatSubmitted` events to:
     - Persist complete chat state including messages, tools, schemas, and references
     - Trigger title generation if enabled and title is empty
     - Update buffer title with relative timestamps

3. **Title Generation System**:
   - Uses the chat's configured LLM adapter for title generation
   - Implements smart content truncation (1000 chars) and prompt engineering
   - Handles title collisions with automatic numbering
   - Updates titles asynchronously using vim.schedule

4. **State Management**:
   - Preserves complete chat context including:
     - Message history with role-based organization
     - Tool states and schemas
     - Reference management
     - Adapter configurations
     - Custom settings

5. **UI Components**:
   - Implements multiple picker interfaces (telescope/default)
   - Provides real-time preview generation with markdown formatting
   - Supports justified text layout for buffer titles
   - Handles window/buffer lifecycle management

6. **Data Flow**:
   - Chat data follows a structured schema (ChatData)
   - Implements proper serialization/deserialization
   - Maintains backward compatibility with existing chats
   - Provides error handling for corrupt or missing data

</details>

## TODOs

- [ ] Add support for additional pickers like snacks, fzf etc

## Related Extensions

- [MCP Hub](https://codecompanion.olimorris.dev/extensions/mcphub.html) extension
- [VectorCode](https://codecompanion.olimorris.dev/extensions/vectorcode.html) extension 

## License

MIT
