# Dynamic AI Assistant - Hardcoded Features That Should Be Dynamic

This guide lists all hardcoded features/functionality/work in the current AI Assistant (press 6) that should be handled dynamically by AI reasoning instead of keyword matching and regex patterns.

## Keyword-Based Command Detection

### File Operations Keywords
- `delete file`, `remove file`, `rm `, `del `
- `rename file`, `move file`, `mv `
- `copy file`, `cp `, `duplicate file`
- `read file`, `show file`, `cat `, `type `, `open file`, `view file`, `display file`
- `write to file`, `write file`, `create file`, `save file`, `make file`
- `create file`, `touch `, `new file`, `make file`
- `append to`, `add to file`
- `file info`, `file size`, `file details`, `stat `

### Directory Operations Keywords
- `delete folder`, `delete directory`, `remove folder`, `remove directory`, `rmdir`, `rd `
- `create folder`, `create directory`, `mkdir`, `make folder`, `make directory`, `new folder`
- `list files`, `show files`, `ls`, `dir`, `list directory`, `show directory`, `what files`, `show contents`
- `go inside`, `gi inside`, `go to`, `navigate to`, `cd `, `change directory`, `open folder`, `enter folder`, `enter directory`
- `pwd`, `current directory`, `where am i`, `current folder`, `working directory`

### Git Operations Keywords
- `switch`, `checkout` (with `branch`)
- `clone` (with `git`, `github`, `repo`, `http`)
- `git` (any git command)
- `git switch`, `git checkout`, `git clone`, `git status`, `git pull`, `git push`, `git commit`, `git add`, `git log`, `git diff`, `git stash`, `git fetch`, `git merge`, `git rebase`, `git remote`, `git branch`

### Build Operations Keywords
- `build` (with `backend` or `compile`)

### Script Execution Keywords
- `run experiment`, `execute experiment`, `start experiment`, `launch experiment`
- `run benchmark`, `execute benchmark`, `start benchmark`, `launch benchmark`
- `run test`, `execute test`, `run tests`, `execute tests`
- `run algorithm`, `execute algorithm`, `run circuit`, `execute circuit`
- `run simulation`, `execute simulation`, `start simulation`
- `run analysis`, `execute analysis`
- `monitor`, `track execution`, `watch execution`
- `.py`, `.sh`, `.ps1`, `.bat`, `.cmd`, `script`

### Terminal/Shell Keywords
- `run`, `execute`, `command`, `terminal`, `shell`, `powershell`, `cmd`, `bash`

### Package Management Keywords
- `install` (with `pip`, `npm`, `package`, `yarn`, `conda`)
- `uninstall` (with `pip`, `npm`, `package`)

### Search Keywords
- `find file`, `search file`, `locate`, `where is`
- `search in`, `grep`, `find text`, `search for`

### Environment Keywords
- `env`, `environment`, `set variable`, `get variable`

### Conversation Keywords
- `hello`, `hi`, `hey`, `greet`
- `backend`, `configure`, `setup`
- `circuit`, `quantum`, `gate`
- `help`, `what`, `how`, `can you`

## Hardcoded Regular Expression Patterns

### File Path Extraction Patterns
- Windows paths: `r'([A-Za-z]:[\\\/][^\s]+)'`
- Unix paths: `r'([\/~][^\s]+)'`
- File extensions: `r'(\S+\.(?:py|sh|ps1|bat|cmd|yaml|yml|json|txt|md|toml))'`
- Relative paths: `r'([./\\]\S+)'`
- Folder patterns: `r'(\w+[/\\]\w+)'`

### Command Extraction Patterns
- Backtick commands: `r'`([^`]+)`'`
- Run/execute patterns: `r'(?:run|execute|command)\s+(.+)'`

### Git Branch Name Patterns
- `r'(?:switch|checkout)\s+(?:to\s+)?git\s+branch\s+([\w\-\.\/]+)'`
- `r'git\s+(?:switch|checkout)\s+([\w\-\.\/]+)'`
- `r'(?:switch|checkout)\s+(?:to\s+)?branch\s+([\w\-\.\/]+)'`
- `r'branch\s+([\w]+-[\w\-\.]+)'`
- `r'branch\s+([\w\-\.\/]+)'`

### Clone Destination Patterns
- `r'(?:in|to|at|into)\s+(?:this\s+)?(?:folder|directory|path|location)?\s*[:\s]+([A-Za-z]:[\\\/][^\s,]+|[\/~][^\s,]+)'`
- Multiple variations with Windows/Unix path handling

### Content Extraction Patterns
- `r'content\s*[=:]\s*["\']?(.+?)["\']?\s*$'`
- `r'with\s+content\s+(.+?)$'`
- `r'(?:write|create|save)\s+(?:to\s+)?(?:file\s+)?["\']?([^\s"\']+)["\']?\s+(?:with\s+)?content\s+(.+)'`

### File Name Patterns
- `r'(?:file\s+)?called\s+([^\s]+\.?\w*)'`
- `r'create\s+(?:a\s+)?file\s+([^\s]+\.?\w*)'`
- `r'make\s+(?:a\s+)?file\s+([^\s]+\.?\w*)'`
- `r'new\s+file\s+([^\s]+\.?\w*)'`

### Subdirectory Navigation Patterns
- `r'go\s+into\s+(?:this\s+)?([a-zA-Z_][a-zA-Z0-9_/\\-]+)(?:\s+directory)'`
- `r'(?:and\s+then\s+)?go\s+into\s+(?:the\s+)?([a-zA-Z_][a-zA-Z0-9_/\\-]+)'`
- `r'navigate\s+(?:to|into)\s+(?:the\s+)?([a-zA-Z_][a-zA-Z0-9_/\\-]+)'`
- `r'cd\s+([a-zA-Z_][a-zA-Z0-9_/\\-]+)'`

### Script Execution Patterns
- `r'run\s+(?:this\s+)?(\S+\.py)'`
- `r'execute\s+(?:this\s+)?(\S+\.py)'`
- `r'(\S+\.py)\s+script'`

### Repository URL Pattern
- `r'(https?://[^\s]+|git@[^\s]+)'`

## Hardcoded Provider Mappings

### LLM Provider Map (Duplicated)
- `'local': 'ollama'`, `'ollama': 'ollama'`
- `'openai': 'openai'`, `'anthropic': 'anthropic'`
- `'google': 'google'`, `'gemini': 'google'`
- `'xai': 'xai'`, `'grok': 'xai'`
- `'deepseek': 'deepseek'`, `'mistral': 'mistral'`
- `'groq': 'groq'`, `'together': 'together'`
- `'openrouter': 'openrouter'`, `'cohere': 'cohere'`
- `'lmstudio': 'lmstudio'`, `'llamacpp': 'llama_cpp'`
- `'perplexity': 'perplexity'`, `'fireworks': 'fireworks'`
- `'huggingface': 'huggingface'`, `'replicate': 'replicate'`
- `'azure_openai': 'azure_openai'`, `'azure': 'azure_openai'`

### Simulation Providers Set
- `'vertex_ai'`, `'aws_bedrock'`, `'ai21'`, `'deepinfra'`

## Hardcoded Backend Names List
- `qiskit`, `cirq`, `pennylane`, `braket`, `pyquil`, `projectq`
- `qsharp`, `quest`, `qulacs`, `lret`, `lret-pennylane`, `lret-cirq`
- `lret-phase`, `lret-phase7`, `lret-phase-7`, `stim`, `pyqrack`
- `quantumsim`, `qibo`, `qsim`, `tensorflow-quantum`, `yao`, `quirk`

## Hardcoded Skip Words Lists

### Branch Name Skip Words
- `the`, `a`, `to`, `into`, `from`, `branch`, `named`, `called`
- `git`, `this`, `that`, `and`, `then`, `go`, `directory`, `folder`
- `run`, `execute`, `script`, `file`, `switch`, `checkout`

### Subdirectory Skip Words
- `git`, `branch`, `switch`, `checkout`, `and`, `then`, `this`, `run`

## Hardcoded Stop Boundaries for Multi-Step Commands
- `and then go`, `then go`, `and go`
- `and then run`, `then run`, `and run`
- `and then execute`, `then execute`, `and execute`
- `and then navigate`, `then navigate`
- `and then open`, `then open`
- `and then cd`, `then cd`
- `and then create`, `then create`
- `and then delete`, `then delete`
- `and then list`, `then list`

## Hardcoded Action Indicators
- `and then`, `then go`, `then run`, `then execute`
- `then switch`, `then navigate`, `then cd`, `then open`

## Hardcoded LLM Intent Extraction Operations
- `FILE_CREATE`, `FILE_READ`, `FILE_WRITE`, `FILE_DELETE`
- `FILE_COPY`, `FILE_MOVE`, `FILE_APPEND`
- `DIR_CREATE`, `DIR_DELETE`, `DIR_LIST`, `DIR_NAVIGATE`
- `GIT_CLONE`, `GIT_STATUS`, `GIT_PULL`, `GIT_PUSH`, `GIT_COMMIT`
- `GIT_ADD`, `GIT_BRANCH`, `GIT_CHECKOUT`, `GIT_LOG`, `GIT_DIFF`
- `TERMINAL_CMD`, `PWD`, `NONE`

## Hardcoded LLM System Prompts

### Intent Extraction Prompt
- Full structured prompt with operation types and examples
- Fixed JSON response format
- Hardcoded confidence threshold (0.6)

### General Chat System Prompt
- Fixed context window (last 5 messages)
- Hardcoded system instructions about being helpful
- Fixed temperature (0.7) and max tokens (1024)

## Hardcoded Response Templates

### Greeting Response
- Fixed "Hello! I'm your AI assistant..." message
- Hardcoded capability list

### Backend Configuration Response
- Fixed step-by-step backend configuration guide
- Hardcoded qubit threshold (<20 qubits)

### Circuit/Quantum Response
- Fixed quantum gate descriptions
- Hardcoded Bell state example

### Help Response
- Fixed agent capabilities message

### Default Response
- Fixed template for unknown queries

## Hardcoded File Size Thresholds
- 1024 bytes for byte/KB threshold
- 1024 * 1024 for KB/MB threshold
- 5000 characters for file read truncation
- 200 characters for content preview truncation
- 1000 characters for script output truncation
- 20 matches for file search limit
- 10 directory entries for listing
- 15 items for pwd contents
- 60 characters for grep line preview

## Hardcoded Timeout Values
- 30 seconds for git fetch
- 30 seconds for git switch/checkout
- 60 seconds for git commands in cd+git
- 60 seconds for terminal commands
- 120 seconds for git clone
- 120 seconds for script execution

## Hardcoded Temperature Values
- 0.1 for intent extraction (low for consistency)
- 0.7 for general chat

## Hardcoded Max Tokens
- 500 for intent extraction
- 1024 for general chat

## Hardcoded Context Window
- Last 5 messages for conversation context

## Hardcoded Common Environment Variables
- `PATH`, `HOME`, `USER`, `SHELL`, `PYTHON`, `VIRTUAL_ENV`

## Hardcoded Package Managers
- `pip` for Python
- `npm` for Node.js
- `yarn` for JavaScript
- `conda` for Conda environments

## Hardcoded File Extensions
- `.py`, `.sh`, `.ps1`, `.bat`, `.cmd` for scripts
- `.yaml`, `.yml`, `.json`, `.txt`, `.md`, `.toml` for config files

## Hardcoded Platform-Specific Logic
- Windows path detection (C:\)
- Unix path detection (/ or ~)
- Shell parameter (True for Windows, False for Unix)

## Hardcoded Shell Commands
- `python` for Python scripts
- `bash` for bash scripts
- `powershell -ExecutionPolicy Bypass -File` for PowerShell scripts

## Hardcoded Sentiment/Intent Words

### Greeting Words
- `hello`, `hi`, `hey`, `greet`

### Backend Words
- `backend`, `configure`, `setup`

### Circuit Words
- `circuit`, `quantum`, `gate`

### Help Words
- `help`, `what`, `how`, `can you`

## Multi-Step Command Detection Logic
- Hardcoded counter for "then" actions (threshold: 2+)
- Fixed sequential step execution order
- Hardcoded step numbering (Step 1, Step 2, etc.)

## Hardcoded Retry Logic
- Git branch switch: 4 different command attempts in sequence
- No dynamic strategy selection

## Hardcoded Path Cleaning Logic
- Fixed rules for removing trailing punctuation
- Hardcoded word removal patterns
- Fixed path separator conversions

## Hardcoded Message Formatting
- Fixed emoji usage (✅, ❌, ⚠️, 📄, 📁, etc.)
- Hardcoded markdown code block syntax
- Fixed message structure templates

## Hardcoded Statistics Tracking
- `repos_cloned`, `commands_run`, `total_requests`, `avg_response_time`
- Fixed stat categories

## Hardcoded Button IDs
- `btn-send`, `btn-stop`, `btn-clear`, `btn-import`, `btn-export`
- `btn-new`, `btn-toggle-sidebar`, `btn-collapse-all`

## Hardcoded Keyboard Shortcuts
- `ctrl+m` for send
- `ctrl+z` for undo
- `ctrl+y` for redo
- `ctrl+n` for new chat
- `ctrl+s` for export
- Number keys 1-6 for navigation

## Hardcoded Conversation Export Format
- JSON with fixed structure
- Fixed timestamp format (ISO)
- Fixed filename pattern with timestamp

## Hardcoded Terminal Shells
- PowerShell for Windows
- Bash for Unix
- Fixed shell detection logic

## Hardcoded Error Messages
- Fixed error message templates
- Hardcoded error categories

## Hardcoded Success Messages
- Fixed success message templates
- Hardcoded status indicators

## Hardcoded Agent Capabilities Message
- Fixed list of capabilities
- Hardcoded examples for each operation type
- Fixed formatting and structure

## Hardcoded Content Pattern Matching
- Multiple regex patterns for content extraction
- Fixed priority order for pattern matching

## Hardcoded File Creation Logic
- Fixed parent directory creation behavior
- Hardcoded exist check response

## Hardcoded Git Command Sequence
- Fixed order: fetch → switch → checkout → checkout -b → switch -c
- No adaptive selection based on context

## Hardcoded Directory Listing Format
- Fixed emoji for files (📄) and folders (📁)
- Hardcoded sorting (sorted())
- Fixed item count limits

## Hardcoded Date/Time Formatting
- ISO format for timestamps
- Fixed datetime patterns for session IDs

## Hardcoded Recent Context Length
- Last 5 messages for LLM context
- Fixed conversation history depth

## Hardcoded JSON Parsing Fallbacks
- Fixed quote replacement (`'` → `"`)
- Hardcoded JSON extraction regex

## Hardcoded Command Normalization
- Fixed rules for backtick extraction
- Hardcoded command prefix patterns

## Hardcoded Build System Detection
- Fixed build system types
- Hardcoded detection patterns

## Hardcoded Permission Checks
- Fixed permission error messages
- Hardcoded access check flags (R_OK, W_OK)

## Hardcoded Path Expansion
- Fixed expanduser and expandvars usage
- Hardcoded environment variable patterns

## Hardcoded Line Number Limits
- 60 characters for grep preview
- 100 characters for file content preview
- 1000 characters for output truncation

## Hardcoded Model/Provider Selection UI
- Fixed dropdown provider list in settings
- Hardcoded provider display names

## Hardcoded Chat Session Structure
- Fixed ChatMessage dataclass fields
- Hardcoded session metadata fields

## Hardcoded UI Component IDs
- `chat-log`, `prompt-input`, `thinking-indicator`, `history-list`
- Fixed CSS class names

## Hardcoded Theme Colors
- Fixed color codes for UI elements
- Hardcoded dark theme palette

## Hardcoded Welcome Message
- Fixed welcome code content
- Hardcoded capability showcase

## Hardcoded Binding Priorities
- Fixed keyboard binding order
- Hardcoded show/hide flags

## Hardcoded Tool Execution Visualization
- Fixed progress indicators
- Hardcoded status display formats
