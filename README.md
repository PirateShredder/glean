# GLEAN - A ToDo Scanner for directories containing md files

A Go program that intelligently scans directories for markdown and text files, extracts action items using Gemini AI, and generates TODO files.

**Features:**
- 🎨 Beautiful terminal UI with branded header
- ⏳ Live progress indicators (spinner animations while processing)
- 🚀 Parallel processing (configurable, defaults to 5 concurrent Gemini sessions)
- 🐛 Comprehensive debug mode for troubleshooting
- 🧠 Custom context support (inline or from file) for tailored AI analysis
- 📦 Project-agnostic design - works with any documentation

## Prerequisites

- Go 1.21 or later
- Gemini CLI installed and accessible via `gemini` command
- macOS (designed for macOS environment)

## Installation

```bash
# Build the program
go build -o todo-scanner

# Optional: Install globally
go install
```

## Usage

The program displays a branded header on startup and shows helpful usage information when run without arguments:

```bash
./todo-scanner

# Output:
# ╔═══════════════════════════════════════════════════════════════╗
# ║                                                               ║
# ║                     TODO SCANNER v1.0                         ║
# ║                                                               ║
# ║         Intelligent AI-Powered Action Item Extractor         ║
# ║                                                               ║
# ╚═══════════════════════════════════════════════════════════════╝
#
# USAGE:
#   todo-scanner -scan [options]    Scan directory and generate TODO.md
#   todo-scanner -gather [options]  Consolidate all TODOs into MASTER_TODO.md
#
# OPTIONS:
#   -scan          Scan current directory for .md/.txt files and extract action items
#   -gather        Gather all auto-generated TODO.md files and consolidate them
#   -ctx <text>    Provide context for AI analysis (inline)
#   -fctx <file>   Provide context for AI analysis from a file
#   --debug        Enable verbose debug output
#   --max-concurrent <int>  Maximum concurrent Gemini sessions (default: 5)
```

### Scan Mode (`-scan`)

Scans the current directory and all subdirectories for `.md` and `.txt` files, processes them in parallel with Gemini, and generates a `TODO.md` file with extracted action items.

```bash
# Basic scan
./todo-scanner -scan

# Scan with inline context
./todo-scanner -scan -ctx "Infrastructure project for renewable energy"

# Scan with context from file
./todo-scanner -scan -fctx ./project-context.txt

# With debug output
./todo-scanner -scan --debug

# Scan with 10 concurrent sessions
./todo-scanner -scan --max-concurrent 10
```

**Context Options:**
- `-ctx <text>`: Provide project context inline as a string
- `-fctx <file>`: Provide project context from a file (useful for longer descriptions)
- Context helps Gemini understand your project domain and extract more relevant action items

**Visual Feedback:**
- **Normal mode**: Shows animated spinner while Gemini processes each file
  - Example: `⠋ Processing README.md with Gemini AI...`
- **Debug mode**: Displays detailed logs instead of spinner for full visibility

**What it does:**
1. Recursively scans all subdirectories
2. Finds all `.md` and `.txt` files (excluding hidden directories and files >100KB)
3. Processes each file individually with Gemini in parallel (max 5 concurrent sessions)
4. Cleans up content (removes excessive whitespace)
5. Sends each file to Gemini with optional custom context
6. Extracts action items in bullet-point format
7. Consolidates all results and writes to `TODO.md` in the current directory

### Gather Mode (`-gather`)

Finds all auto-generated `TODO.md` files (created by this tool) and consolidates them into a `MASTER_TODO.md` with deduplicated action items.

```bash
./todo-scanner -gather

# With debug output
./todo-scanner -gather --debug
```

**What it does:**
1. Recursively scans all subdirectories
2. Finds all `TODO.md` files created by this tool
3. Consolidates all action items
4. Sends to Gemini for deduplication and summary
5. Writes results to `MASTER_TODO.md` in the current directory

### Debug Mode (`--debug`)

Enable verbose debug output to see detailed information about what the program is doing at each step.

```bash
./todo-scanner -scan --debug
./todo-scanner -gather --debug
```

**Debug output includes:**
- File discovery and filtering decisions
- File sizes and content lengths
- Gemini API call timing and progress
- Parallel processing status (which file is being processed)
- Error details and warnings
- File I/O operations

## Features

### User Experience
- **Branded Header**: Beautiful ASCII art header displays on startup and help screens
- **Live Progress Indicators**: Animated spinner shows real-time processing status
  - Automatically disabled in debug mode to avoid interference with logs
  - Unicode spinner characters for smooth animation
- **Smart UI**: Spinner messages show which file is being processed

### Performance & Processing
- **Parallel Processing**: Processes up to 5 files concurrently with Gemini for faster execution
- **Smart Filtering**: Skips hidden directories (`.git`, `.cache`), binary files, and files >100KB
- **Content Cleanup**: Removes excessive whitespace and blank lines
- **Individual File Processing**: Each file gets its own Gemini session for better accuracy
- **Non-Blocking**: Uses stdin for Gemini input to prevent hanging

### Intelligence & Accuracy
- **Context-Aware**: Optionally provide custom project context via `-ctx` or `-fctx` for tailored AI analysis
- **Project-Agnostic**: Works with any documentation - from infrastructure projects to software development
- **Auto-Generated Headers**: Marks generated files so `-gather` mode can identify them

### Developer Tools
- **Debug Mode**: Comprehensive logging for troubleshooting and visibility
- **Timing Information**: Shows how long each Gemini call takes (in debug mode)

## Custom Context

The tool supports providing custom context to help Gemini understand your specific project domain and extract more relevant action items.

### Why Use Context?

Without context, Gemini analyzes documents generically. With context, it can:
- Understand project-specific terminology
- Identify domain-relevant action items
- Prioritize tasks based on your project type
- Recognize technology stack or industry-specific TODOs

### How to Provide Context

**Option 1: Inline Context (`-ctx`)**
```bash
./todo-scanner -scan -ctx "React/TypeScript web application for healthcare"
```
Best for: Short, simple project descriptions

**Option 2: Context from File (`-fctx`)**
```bash
./todo-scanner -scan -fctx ./project-context.txt
```
Best for: Detailed project descriptions, multi-line context, reusable context

**Example Context File:**
```
Project: Enterprise SaaS Platform
Technology: Python/Django backend, React frontend, PostgreSQL database
Focus Areas:
- Security and compliance (HIPAA, SOC 2)
- API design and documentation
- Infrastructure automation
- Performance optimization
```

### Context Tips

- Be specific about your project type and domain
- Mention key technologies or frameworks
- Highlight focus areas (e.g., security, performance, documentation)
- Keep it concise but informative (1-3 paragraphs is ideal)

## Example Workflow

### Basic Usage
```bash
# Navigate to a project directory
cd ~/my-project/docs

# Scan for action items
./todo-scanner -scan

# Review the generated TODO.md
cat TODO.md
```

### With Project Context
```bash
# Create a context file for your project
cat > project-context.txt << EOF
This is a renewable energy GPU compute infrastructure project.
We're building modular data centers powered by solar energy.
Focus on technical implementation, vendor coordination, and regulatory compliance.
EOF

# Scan with context
./todo-scanner -scan -fctx ./project-context.txt

# Or use inline context
./todo-scanner -scan -ctx "E-commerce platform using React and Node.js"
```

### Consolidating Multiple TODOs
```bash
# From parent directory, consolidate all TODOs
cd ~/my-project
./todo-scanner -gather

# Review the master TODO
cat MASTER_TODO.md
```

### Debug Mode for Troubleshooting
```bash
# See detailed step-by-step processing
./todo-scanner -scan --debug -ctx "Software development project"
```

## Performance

The parallel processing significantly improves performance:

- **Old approach**: Process 10 files sequentially → ~60-90 seconds
- **New approach**: Process 10 files in parallel (5 concurrent) → ~20-30 seconds

Processing time varies based on:
- File size and content complexity
- Number of files
- Gemini API response time
- Network latency

## Troubleshooting

### Program hangs or doesn't return

The updated version fixes the hanging issue by:
- Using stdin instead of command-line arguments for Gemini
- Processing files individually instead of one large blob
- Running Gemini sessions in parallel with proper goroutine management

If you still experience hanging:
1. Run with `--debug` to see where it's getting stuck
2. Check if `gemini` command is accessible: `which gemini`
3. Test Gemini manually: `echo "test prompt" | gemini`

### No output or empty TODO files

- Ensure files contain actual content (not just whitespace)
- Run with `--debug` to see which files are being processed
- Check that Gemini is returning responses: `echo "List action items" | gemini`

## Notes

- Only one mode (`-scan` or `-gather`) can be used at a time
- Cannot use both `-ctx` and `-fctx` simultaneously - choose one context method
- Context is optional - the tool works fine without it for generic action item extraction
- The `-gather` mode only processes `TODO.md` files created by this tool (identified by header comment)
- Files are processed in parallel (up to 5 concurrent Gemini sessions)
- Use `--debug` for detailed visibility into the processing pipeline
- Context applies to both `-scan` and `-gather` modes
