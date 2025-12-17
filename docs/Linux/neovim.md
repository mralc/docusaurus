# Getting Started with Neovim

## Overview

Neovim is a modern, highly extensible text editor that builds upon the foundation of Vim. It's designed to be faster, more maintainable, and more customizable than traditional Vim, while maintaining compatibility with most Vim plugins and configurations.

**Important:** Don't install Neovim from the Ubuntu repositories as they often contain outdated versions. Instead, install directly from the official Neovim releases to get the latest features and improvements.

## Why Neovim?

Neovim offers several advantages over traditional Vim:

- **Modern Architecture:** Built-in support for asynchronous plugins
- **Better Performance:** Improved responsiveness and speed
- **Lua Configuration:** Native Lua scripting alongside Vimscript
- **Active Development:** Regular updates and new features
- **Built-in LSP:** Native Language Server Protocol support
- **Better Defaults:** Sensible default settings out of the box

## Installation

### Installing from Official Source

To get the latest version of Neovim, download it from the official GitHub releases:

```bash
# Download the latest stable release
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux64.tar.gz

# Extract the archive
tar xzvf nvim-linux64.tar.gz

# Move to a system location
sudo mv nvim-linux64 /opt/nvim

# Create a symbolic link
sudo ln -s /opt/nvim/bin/nvim /usr/local/bin/nvim
```

### Verify Installation

Check that Neovim is installed correctly:

```bash
nvim --version
```

You should see version 0.9.0 or higher.

## Configuration with Kickstart.nvim

The easiest way to get started with a solid Neovim configuration is to use [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim). This provides a well-documented, minimal starting point that you can customize to your needs.

### Install Kickstart.nvim

```bash
# Backup your existing config (if any)
mv ~/.config/nvim ~/.config/nvim.backup

# Clone kickstart.nvim
git clone https://github.com/nvim-lua/kickstart.nvim.git ~/.config/nvim
```

### Install Required Dependencies

Kickstart.nvim requires several tools for full functionality:

```bash
# Install ripgrep (for fast file searching)
sudo apt-get install ripgrep

# Install fd (optional but recommended for faster file finding)
sudo apt-get install fd-find

# Install a Nerd Font for icons (optional)
# Download from https://www.nerdfonts.com/
```

### First Launch

When you first launch Neovim after installing kickstart.nvim:

```bash
nvim
```

It will automatically:
1. Install the plugin manager (lazy.nvim)
2. Download and install all configured plugins
3. Set up Language Server Protocol (LSP) servers

This may take a few minutes on the first run.

## Essential Keybindings and Commands

### File Navigation

**`:Ex`** - Open Netrw (Neovim's built-in file explorer)
- Navigate files and directories in a tree-like view
- Press `Enter` to open files
- Press `-` to go up a directory
- Press `%` to create a new file
- Press `d` to create a new directory

**`:Telescope`** - Open Telescope fuzzy finder
- `<leader>ff` - Find files in your project
- `<leader>fg` - Live grep (search text in files)
- `<leader>fb` - Browse open buffers
- `<leader>fh` - Search help tags

### Basic Navigation

- `h, j, k, l` - Move left, down, up, right
- `w` - Jump to start of next word
- `b` - Jump to start of previous word
- `0` - Jump to start of line
- `$` - Jump to end of line
- `gg` - Jump to start of file
- `G` - Jump to end of file

### Editing

- `i` - Enter insert mode before cursor
- `a` - Enter insert mode after cursor
- `o` - Open new line below and enter insert mode
- `O` - Open new line above and enter insert mode
- `dd` - Delete current line
- `yy` - Yank (copy) current line
- `p` - Paste after cursor
- `u` - Undo
- `Ctrl-r` - Redo

### Searching

- `/pattern` - Search forward for pattern
- `?pattern` - Search backward for pattern
- `n` - Jump to next match
- `N` - Jump to previous match
- `*` - Search for word under cursor

### Split Windows

- `:split` or `:sp` - Split horizontally
- `:vsplit` or `:vs` - Split vertically
- `Ctrl-w h/j/k/l` - Navigate between splits
- `Ctrl-w =` - Make splits equal size

## Language Server Protocol (LSP)

Kickstart.nvim comes with LSP support configured. To install language servers:

```bash
# Inside Neovim, for a specific language
:LspInstall

# The LSP will provide:
# - Code completion
# - Go to definition
# - Find references
# - Inline diagnostics
# - Code actions
```

Common LSP keybindings (configured in kickstart.nvim):
- `gd` - Go to definition
- `gr` - Go to references
- `K` - Show hover documentation
- `<leader>rn` - Rename symbol
- `<leader>ca` - Code actions

## Customizing Your Configuration

The beauty of kickstart.nvim is that it's fully documented. To customize:

1. Open the main config file:
   ```bash
   nvim ~/.config/nvim/init.lua
   ```

2. Read through the comments - they explain every section
3. Modify settings to your preference
4. Add new plugins in the appropriate sections
5. Restart Neovim or run `:source %` to reload

## Essential Plugins Included

Kickstart.nvim includes several essential plugins:

- **Telescope** - Fuzzy finder for files, text, and more
- **Treesitter** - Advanced syntax highlighting
- **LSP Config** - Language server protocol integration
- **nvim-cmp** - Autocompletion engine
- **Which-key** - Displays available keybindings
- **Gitsigns** - Git integration and indicators

## Tips for Learning

1. **Start Small:** Don't try to learn everything at once
2. **Use Vimtutor:** Run `vimtutor` in your terminal for an interactive tutorial
3. **Learn One Thing Daily:** Master one new command or motion each day
4. **Use Which-key:** Press `<leader>` and wait - it shows available commands
5. **Read the Docs:** Use `:help` followed by any command to learn more

## Troubleshooting

### Plugin Issues

If plugins aren't working:
```bash
# Inside Neovim
:Lazy sync
```

### LSP Not Working

```bash
# Check LSP status
:LspInfo

# Restart LSP server
:LspRestart
```

### Reset Configuration

If something breaks, you can always start fresh:
```bash
# Remove current config
rm -rf ~/.config/nvim

# Restore backup or reinstall kickstart
git clone https://github.com/nvim-lua/kickstart.nvim.git ~/.config/nvim
```

## Resources

- [Official Neovim Documentation](https://neovim.io/doc/)
- [Kickstart.nvim Repository](https://github.com/nvim-lua/kickstart.nvim)
- [Awesome Neovim](https://github.com/rockerBOO/awesome-neovim) - Curated list of plugins
- [r/neovim](https://reddit.com/r/neovim) - Active community

## Conclusion

Neovim is a powerful editor that rewards the time invested in learning it. With kickstart.nvim, you get a solid foundation that you can build upon as you become more comfortable. Start with the basics, customize gradually, and enjoy the journey to becoming a more efficient developer!