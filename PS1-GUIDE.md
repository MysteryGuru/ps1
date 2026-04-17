# PS1 Guide: Customize Your Bash Prompt

A complete beginner-friendly guide to understanding and creating custom shell prompts using PS1.

---

## Table of Contents

1. [What is PS1?](#what-is-ps1)
2. [Basic Concepts](#basic-concepts)
3. [Understanding Backslash-Escaped Sequences](#backslash-sequences)
4. [Adding Colors to Your Prompt](#adding-colors)
5. [Foreground vs Background Colors](#foreground-vs-background)
6. [The Critical Wrapping Rule (Fixing Text Overlap)](#the-critical-wrapping-rule)
7. [Common Mistakes](#common-mistakes)
8. [Building Your Custom Prompt](#building-your-custom-prompt)
9. [PROMPT_COMMAND: Dynamic Prompts](#prompt_command-dynamic-prompts)
10. [Quick Color Palettes (Copy & Paste Ready)](#quick-color-palettes-copy--paste-ready)
11. [Testing Colors Safely](#testing-colors-safely)
12. [Undoing a Broken Prompt](#undoing-a-broken-prompt)
13. [Multi-Line Prompts](#multi-line-prompts)
14. [Environment Variables That Affect Colors](#environment-variables-that-affect-colors)
15. [Making Changes Permanent](#making-changes-permanent)
16. [Secondary Prompts (PS2, PS3, PS4)](#secondary-prompts-ps2-ps3-ps4)
17. [Using PS1 with Zsh](#using-ps1-with-zsh)
18. [One-Liners for Quick Customization](#one-liners-for-quick-customization)

---

## What is PS1?

**PS1** is a special environment variable in Bash that controls what your command prompt looks like.

When Bash is waiting for you to type a command, it prints the value of `PS1` and then waits for your input.

### Quick Example

```bash
# This sets your prompt to show just the current directory and a ">" symbol
PS1="\W > "
```

---

## Basic Concepts

### How to Change Your Prompt

Use the `export` command to change PS1:

```bash
export PS1="new prompt here"
```

**Important:** This change only lasts for the current terminal session. When you open a new terminal, it reverts to the default.

### Parameter Expansion

When you type commands inside PS1, Bash expands variables. Use **double quotes** to allow this to happen:

```bash
# Double quotes: Variables and escapes WILL be expanded
PS1="Hello: \u@\h"

# Single quotes: Variables and escapes will NOT be expanded (too literal)
PS1='Hello: \u@\h'     # This would print "\u@\h" literally!
```

---

## Backslash-Escaped Sequences

These special sequences expand to show system information. Use them inside **double-quoted** PS1:

| Sequence | What It Shows |
|----------|---------------|
| `\u` | Current username |
| `\h` | Hostname (short) |
| `\H` | Hostname (full) |
| `\w` | Full working directory (with `~` for home) |
| `\W` | Current directory name only (basename) |
| `\t` | Current time (HH:MM:SS) |
| `\@` | Current time (12-hour with AM/PM) |
| `\n` | New line |
| `\s` | Shell name |
| `$` | `#` if root, `$` otherwise |

### Examples

```bash
# Show username@host:/current/directory$
PS1="\u@\h:\W$ "

# Show just the directory name and prompt
PS1="\W > "

# Show with time
PS1="[\t] \u@\h:\W$ "
```

---

## Adding Colors to Your Prompt

### What Are ANSI Color Codes?

Terminals understand special **escape sequences** that tell them to change color. These start with `\e[` (or `\033[`) and end with `m`.

### Standard 8-Color Codes (Easiest)

```bash
# Foreground (text) colors:
30=black, 31=red, 32=green, 33=yellow, 34=blue, 35=magenta, 36=cyan, 37=white

# Background colors (add 10):
40=black bg, 41=red bg, 42=green bg, 43=yellow bg, etc.

# Reset (very important!):
0=reset all colors
```

### Simple Color Example

```bash
# Red text
PS1="\e[31mHello\e[0m"       # "Hello" in red

# Green background
PS1="\e[42mDirectory\e[0m"   # "Directory" with green background

# Red text on yellow background
PS1="\e[31;43mText\e[0m"     # Combine with semicolon: 31 (red text) + 43 (yellow bg)
```

### Truecolor (24-bit RGB)

Modern terminals support full RGB color using 24 bits (16.7 million colors):

```
Foreground: \e[38;2;R;G;Bm   (where R, G, B are 0-255)
Background: \e[48;2;R;G;Bm
```

**Example: Orange text**
```bash
\e[38;2;255;140;0m
```

**Example: Dark blue background**
```bash
\e[48;2;10;20;60m
```

---

## Foreground vs Background

- **Foreground** = the color of the text itself
- **Background** = the color of the area behind the text

```bash
# Red text (foreground)
\e[31m

# Red background, white text (foreground + background)
\e[41m\e[97m
```

### Visual Demo

```
Without colors:
user@host:~/directory$ echo "hello"

With foreground color (text is red):
\e[31muser@host:~/directory\e[0m$ echo "hello"

With background color (area behind text is red):
\e[41muser@host:~/directory\e[0m$ echo "hello"

With both (red text on yellow background):
\e[31;43muser@host:~/directory\e[0m$ echo "hello"
```

---

## The Critical Wrapping Rule ⚠️

### The Problem: Text Overlap

When you try to type a long command, does the text **overwrite your prompt** instead of wrapping to the next line?

```
user@directory$ this is a long command that wraps
my long command startshere instead of wrapping!
```

This happens because Bash miscalculates the visible length of your prompt when it contains color codes.

### The Solution: Wrap Non-Printing Sequences

**All escape sequences (color codes) must be wrapped with `\[` and `\]`:**

```bash
# WRONG (causes text overlap):
PS1="\e[31m\u@\h:\W\e[0m$ "

# CORRECT (no overlap):
PS1="\[\e[31m\]\u@\h:\W\[\e[0m\]$ "
```

### Why This Works

- `\[` tells Bash: "Start a non-printing sequence"
- `\]` tells Bash: "End a non-printing sequence"
- This lets Bash correctly calculate prompt length for line wrapping

**Rule of Thumb:** Wrap every escape sequence (starts with `\e[` or `\033[`) with `\[` and `\]`.

---

## Common Mistakes

### ❌ Mistake 1: Forgetting to Wrap Color Codes

```bash
# WRONG - causes text overlap
PS1="\e[31m\u@\h\e[0m$ "

# RIGHT - wraps non-printing sequences
PS1="\[\e[31m\]\u@\h\[\e[0m\]$ "
```

### ❌ Mistake 2: Wrong Truecolor Format

```bash
# WRONG - incorrect SGR format
PS1="\e[48;255;0;0m"    # Not enough semicolons!

# RIGHT - correct truecolor background format
PS1="\[\e[48;2;255;0;0m\]"   # 48;2;R;G;B
```

### ❌ Mistake 3: Not Resetting Colors

If you don't reset colors at the end (`\e[0m`), colors will leak into your typed commands:

```bash
# WRONG - colors leak
PS1="\[\e[31m\]\u@\h$ "              # Text stays red!

# RIGHT - colors reset
PS1="\[\e[31m\]\u@\h\[\e[0m\]$ "    # Text returns to normal after prompt
```

### ❌ Mistake 4: Using Single Quotes

Single quotes prevent variable expansion:

```bash
# WRONG - nothing gets expanded
PS1='\u@\h:\W$ '          # Prints literal "\u@\h:\W"

# RIGHT - use double quotes
PS1="\u@\h:\W$ "          # Prints "user@host:directory"
```

---

## Building Your Custom Prompt

Let's build the example from this guide:

```bash
PS1="[\e[48;2;255;0;0m] \W [\e[48;2;0;200;0m] > [\e[0m] "
```

### Step-by-Step Breakdown

1. `[` - Literal bracket
2. `\e[48;2;255;0;0m` - **Red background** (RGB: 255, 0, 0)
3. `]` - Literal bracket
4. `\W` - Current directory name (e.g., "projects")
5. `[` - Literal bracket
6. `\e[48;2;0;200;0m` - **Green background** (RGB: 0, 200, 0)
7. `]` - Literal bracket
8. `>` - Prompt symbol
9. `[` - Literal bracket
10. `\e[0m` - **Reset colors** (important!)
11. `]` - Literal bracket

### Properly Wrapped Version (Correct!)

```bash
PS1="[\[\e[48;2;255;0;0m\]] \W [\[\e[48;2;0;200;0m\]] > [\[\e[0m\]] "
```

**Or with better spacing:**

```bash
PS1="\[\e[48;2;255;0;0m\] \W \[\e[48;2;0;200;0m\] > \[\e[0m\] "
```

### More Example Prompts

**Simple colored directory:**
```bash
PS1="\[\e[1;32m\]\W\[\e[0m\]$ "   # Bold green directory, then $
```

**With username, host, and time:**
```bash
PS1="\[\e[1;34m\]\u@\h\[\e[0m\]:\[\e[1;32m\]\W\[\e[0m\]$ "
```

**Minimal:**
```bash
PS1="> "
```

**Professional:**
```bash
PS1="\[\e[1;33m\][\t]\[\e[0m\] \u:\W$ "   # Shows time in yellow
```

---

## PROMPT_COMMAND: Dynamic Prompts

`PROMPT_COMMAND` is a special variable that runs a command **before** your prompt appears each time. Use it to update information dynamically without slowing down the prompt itself.

### Why Use PROMPT_COMMAND?

If you put complex commands inside PS1 (like getting git branch), it runs every time the prompt displays, which can be slow. `PROMPT_COMMAND` runs expensive operations once and stores the result in a variable that PS1 prints.

### Example: Git Branch in Your Prompt

```bash
# This runs once per prompt and stores the git branch
PROMPT_COMMAND='GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)'

# Then PS1 just prints the stored variable (fast!)
PS1='\u@\h:\W${GIT_BRANCH:+ ($GIT_BRANCH)}$ '
```

What this does:
- `GIT_BRANCH=$(...)` — Gets the current git branch
- `2>/dev/null` — Silences error messages if not in a git repo
- `${GIT_BRANCH:+ ($GIT_BRANCH)}` — Prints " (branch-name)" only if GIT_BRANCH is set

### Example: Update Window Title

```bash
# Update the terminal window title with the current directory
PROMPT_COMMAND='echo -ne "\033]0;${PWD##*/}\007"'
```

### Combining Multiple Commands in PROMPT_COMMAND

```bash
# Separate with semicolons (don't overwrite existing PROMPT_COMMAND)
PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND; }new_command"

# Or use a function
my_prompt_update() {
    GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)
    # ... other updates
}
PROMPT_COMMAND=my_prompt_update
```

---

## Quick Color Palettes (Copy & Paste Ready)

Here are pre-made color schemes you can copy directly into your terminal:

### 🔴 Red & Dark (Professional)

```bash
export PS1="\[\e[48;2;180;0;0m\]\[\e[38;2;255;255;255m\] \W \[\e[0m\] "
```

### 🟢 Green & Black (Terminal-Style)

```bash
export PS1="\[\e[48;2;0;100;0m\]\[\e[38;2;0;255;0m\] \W \[\e[0m\]$ "
```

### 🔵 Blue & Light (Clean)

```bash
export PS1="\[\e[48;2;25;118;210m\]\[\e[38;2;255;255;255m\] \W \[\e[0m\] "
```

### 🟣 Purple & Cream (Eye-Friendly)

```bash
export PS1="\[\e[48;2;88;68;132m\]\[\e[38;2;240;240;240m\] \W \[\e[0m\] "
```

### 🟡 Orange & Dark (Warm)

```bash
export PS1="\[\e[48;2;255;140;0m\]\[\e[38;2;0;0;0m\] \W \[\e[0m\] "
```

### 🔲 Minimal (Just Directory with Color)

```bash
export PS1="\[\e[1;35m\]\W\[\e[0m\]$ "
```

### 🎨 Multi-Section (Your Example)

```bash
export PS1="\[\e[48;2;255;0;0m\] \W \[\e[48;2;0;200;0m\] > \[\e[0m\] "
```

---

## Testing Colors Safely

Before you break your prompt, test colors first!

### Test Foreground Color

```bash
# Test if a color code works
printf "\e[38;2;255;100;0mThis is orange text\e[0m\n"

# If you see orange text, the code is correct
```

### Test Background Color

```bash
printf "\e[48;2;255;100;0mThis has orange background\e[0m\n"
```

### Test Your Full PS1 (Before Committing)

```bash
# Set it temporarily to test
PS1="\[\e[48;2;255;100;0m\]test\[\e[0m\] "

# Type something long to verify wrapping works
echo "This is a very long line to check if wrapping works correctly"

# If wrapping breaks, fix it and try again
# Once you're happy, add it to ~/.bashrc
```

---

## Undoing a Broken Prompt

Accidentally made a prompt that looks terrible? Here's how to fix it:

### Immediate Reset (Current Session Only)

```bash
# Clear any color bleeding
echo -e '\e[0m'

# Restore to default prompt
PS1='\u@\h:\W\$ '

# Or export to make it stick for subshells
export PS1='\u@\h:\W\$ '
```

### Restore Original Default

```bash
# Most systems default to this:
PS1='\u@\h:\W\$ '

# Or go back to whatever was in ~/.bashrc before
source ~/.bashrc
```

### Completely Reset Everything

```bash
# This command clears the terminal and all color codes
reset
```

---

## Multi-Line Prompts

You can use `\n` to split your prompt across multiple lines:

### Two-Line Prompt (Directory on Top, Prompt Below)

```bash
export PS1="\[\e[1;32m\]\u@\h:\W\[\e[0m\]\n\[\e[1;33m\]>\[\e[0m\] "
```

Shows:
```
user@host:directory
> 
```

### Info Line + Command Prompt

```bash
export PS1="\t \u@\h\n\[\e[1;31m\]\W\[\e[0m\]$ "
```

Shows:
```
14:30:45 user@host
mydir$
```

---

## Environment Variables That Affect Colors

Your terminal's behavior with colors depends on these variables:

| Variable | What It Does | How to Check |
|----------|--------------|--------------|
| `TERM` | Tells Bash what kind of terminal you have | `echo $TERM` |
| `COLORTERM` | Indicates if terminal supports 24-bit color | `echo $COLORTERM` |
| `NO_COLOR` | Some apps disable colors if this is set | `echo $NO_COLOR` |

### Check if Your Terminal Supports Truecolor

```bash
# Test this command
printf "\x1b[38;2;255;0;0mRED TEXT\x1b[0m\n"

# If you see RED in actual red color, truecolor works
# If the text looks corrupted, you might need 8-color codes instead
```

---

## Making Changes Permanent

Changes made with `export PS1=...` only last for the current terminal session.

### Option 1: Add to Bash Config File

Edit `~/.bashrc` and add your PS1 line:

```bash
# Open the file
nano ~/.bashrc

# Add at the end:
export PS1="\[\e[1;32m\]\W\[\e[0m\]$ "

# Save (Ctrl+O, Enter, Ctrl+X)
```

Then reload your shell:
```bash
source ~/.bashrc
```

### Option 2: Add to Profile File (Runs Once at Login)

Edit `~/.bash_profile`:

```bash
nano ~/.bash_profile

# Add:
export PS1="\[\e[1;32m\]\W\[\e[0m\]$ "
```

### Which File to Use?

- **~/.bashrc** — Runs for every new interactive shell (most common)
- **~/.bash_profile** — Runs only at login
- **~/.profile** — Used by other shells (sh, ksh)

For most users, edit `~/.bashrc`.

---

## Testing Your Changes

Before adding to `~/.bashrc`, test in your current terminal:

```bash
# Test your PS1
PS1="\[\e[1;31m\]\W\[\e[0m\]$ "

# Type a long command to make sure text doesn't overlap:
echo "This is a very long command to test that the text wraps correctly"

# If wrapping works, add it to ~/.bashrc
```

---

## Troubleshooting

### Colors Are Leaking Into Commands

**Problem:** After your prompt, everything is still colored.

**Solution:** Make sure you end with `\[\e[0m\]` (reset).

```bash
# FIX:
PS1="\[\e[31m\]\W\[\e[0m\]$ "
#                       ↑ Add reset before final $
```

### Text Overlaps When Typing Long Commands

**Problem:** Your prompt is overwriting what you type.

**Solution:** Wrap ALL escape sequences with `\[` and `\]`.

```bash
# WRONG:
PS1="\e[31m\W\e[0m$ "

# RIGHT:
PS1="\[\e[31m\]\W\[\e[0m\]$ "
#     ↑           ↑
```

### Colors Won't Show

**Problem:** Your terminal doesn't support colors.

**Solution:** Check if your terminal supports 24-bit color (truecolor):

```bash
# Test with this:
printf "\x1b[38;2;255;0;0mRED\x1b[0m\n"

# If you see red text, truecolor works
# If not, try standard 8-color codes instead:
PS1="\[\e[31m\]\W\[\e[0m\]$ "
```

---

## Quick Reference Card

| Task | Code |
|------|------|
| Show current dir | `\W` |
| Show username | `\u` |
| Show hostname | `\h` |
| Show time | `\t` or `\@` |
| Red text | `\[\e[31m\]` |
| Green text | `\[\e[32m\]` |
| Red background | `\[\e[48;2;255;0;0m\]` |
| Green background | `\[\e[48;2;0;200;0m\]` |
| Reset colors | `\[\e[0m\]` |
| Bold | `\[\e[1m\]` |
| Dim | `\[\e[2m\]` |

---

## Secondary Prompts (PS2, PS3, PS4)

PS1 is the main prompt, but Bash has other prompt variables for different situations:

### PS2: Continuation Prompt (When a Command Wraps)

PS2 appears when you have an incomplete command that spans multiple lines:

```bash
# This sets PS2 to show three dots
export PS2="... "

# When you type an incomplete command:
$ echo "This is a \
... long message"
```

### PS3: Select Menu Prompt

PS3 appears when you use the `select` command in a script:

```bash
export PS3="Choose an option: "

# In a script:
select option in "Option 1" "Option 2" "Exit"; do
  case $option in
    "Option 1") echo "You chose 1" ;;
    "Option 2") echo "You chose 2" ;;
    "Exit") break ;;
  esac
done
```

### PS4: Debug Prompt

PS4 appears when you run a script with debugging enabled (`bash -x script.sh`):

```bash
# Makes debugging output easier to read
export PS4="[DEBUG] + "
```

---

## Using PS1 with Zsh

If you use **Zsh** instead of Bash, you need to use `PROMPT` instead of `PS1`.

Zsh uses different escape sequences:

| Bash | Zsh | Meaning |
|------|-----|---------|
| `\u` | `%n` | Username |
| `\h` | `%M` | Hostname |
| `\W` | `%1~` | Current directory (basename) |
| `\w` | `%~` | Full directory (with ~) |
| `\t` | `%T` | Time |

### Zsh Example (Equivalent to Bash PS1)

```bash
# Bash version
PS1="\[\e[1;32m\]\u@\h:\W\[\e[0m\]$ "

# Zsh version
PROMPT="%B%F{green}%n@%M:%1~%f%b%# "
```

### Adding Colors in Zsh

```bash
# Simple colors without wrapping markers
PROMPT="%F{red}%n@%m:%1~%f%# "

# %F{color} = foreground, %f = reset foreground
# %B = bold, %b = reset bold

# Available colors: black, red, green, yellow, blue, magenta, cyan, white
```

---

## Summary

1. **PS1** is the variable that controls your prompt
2. **Use double quotes** (`"`) so that backslash sequences expand
3. **Always wrap color codes** with `\[` and `\]` to prevent text overlap
4. **Always end with reset** (`\[\e[0m\]`) to prevent color leaking
5. **Use truecolor format** `\e[48;2;R;G;B\m` for backgrounds, `\e[38;2;R;G;B\m` for text
6. **Test first**, then add to `~/.bashrc` to make permanent

Now go customize your prompt! 🎨

---

## One-Liners for Quick Customization

Copy and paste any of these directly into your terminal to instantly change your prompt:

```bash
# Minimal (just the symbol)
PS1='$ '

# Minimal directory
PS1='\W$ '

# Classic username@host:directory
PS1='\u@\h:\W$ '

# Directory with bold green
export PS1="\[\e[1;32m\]\W\[\e[0m\]$ "

# Colored prompt (red directory, blue prompt symbol)
export PS1="\[\e[31m\]\W\[\e[0m\] \[\e[34m\]>\[\e[0m\] "

# With time (yellow)
export PS1="\[\e[1;33m\][\t]\[\e[0m\] \u:\W$ "

# Fancy multi-color (red bg, green prompt)
export PS1="\[\e[48;2;255;0;0m\] \W \[\e[0m\] \[\e[48;2;0;200;0m\]>\[\e[0m\] "

# Simple two-line prompt
export PS1="\u@\h:\W\n\[\e[1;31m\]>\[\e[0m\] "

# Bright cyan text on dark background
export PS1="\[\e[48;2;30;30;30m\]\[\e[38;2;0;255;255m\] \W \[\e[0m\]$ "

# Git branch in prompt
PROMPT_COMMAND='GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)'
export PS1='\u@\h:\W${GIT_BRANCH:+ ($GIT_BRANCH)}$ '
```

**Remember:** These are only temporary. To make them permanent, add to `~/.bashrc`.

---

## Resources

- `man bash` — Search for "PROMPTING" section
- `man echo` — For testing colors with `echo -e`
- [ANSI Code Reference](https://en.wikipedia.org/wiki/ANSI_escape_code)
