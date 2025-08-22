# Neovim Installation Guide for tree-sitter-authzed

This guide provides instructions for installing the Authzed tree-sitter parser in Neovim.

## Issue Summary

The automatic installation via `:TSInstall authzed` has compatibility issues with Neovim's automated compilation process. The manual installation method works reliably.

## Manual Installation (Recommended)

### Prerequisites
- `tree-sitter-cli` installed
- C compiler (gcc/clang)
- Neovim with nvim-treesitter plugin

### Steps

1. **Clone and compile the parser:**
   ```bash
   git clone https://github.com/noamsto/tree-sitter-authzed.git
   cd tree-sitter-authzed
   
   # Generate and compile
   tree-sitter generate
   cc -fPIC -c -I src src/parser.c -o parser.o
   cc -shared parser.o -o authzed.so
   ```

2. **Install parser and queries:**
   ```bash
   # Copy parser binary
   cp authzed.so ~/.local/share/nvim/lazy/nvim-treesitter/parser/
   
   # Copy query files
   mkdir -p ~/.local/share/nvim/lazy/nvim-treesitter/queries/authzed
   cp queries/*.scm ~/.local/share/nvim/lazy/nvim-treesitter/queries/authzed/
   ```

3. **Configure Neovim treesitter:**
   Add to your treesitter configuration:
   ```lua
   local parser_config = require("nvim-treesitter.parsers").get_parser_configs()
   
   parser_config.authzed = {
     install_info = {
       url = "https://github.com/noamsto/tree-sitter-authzed",
       files = { "src/parser.c" },
       branch = "main",
     },
     filetype = "authzed",
     maintainers = { "@noamsto" },
   }
   
   -- Set up filetype detection
   vim.filetype.add({
     extension = {
       authzed = "authzed",
       zed = "authzed",
     },
   })
   ```

4. **Restart Neovim** and open any `.zed` or `.authzed` file.

## Automatic Installation Issues

### Attempted Solutions
- ✅ Repository configuration correct
- ✅ Grammar.js includes latest `.all()` and `.any()` method support  
- ✅ Queries compatible with Neovim treesitter
- ❌ `requires_generate_from_grammar = true` - causes installation failures
- ❌ Adding to `ensure_installed` list - parser not installed

### Root Causes Identified
1. **Compilation Environment**: Neovim's automated compilation uses different flags/environment than manual compilation
2. **Tree-sitter Version Mismatch**: Package.json specifies `^0.20.8` but system has `0.25.6`
3. **Build Process**: Neovim treesitter's build process has compatibility issues with the grammar setup

### Error Symptoms
- `:TSInstall authzed` appears to succeed but no parser is actually installed
- No `authzed.so` file created in parser directory
- No `authzed.revision` file in parser-info directory
- "Parser could not be created for buffer X and language 'authzed'" errors

## Verification

After installation, verify syntax highlighting works:

1. Create test file `test.zed`:
   ```
   definition user {}
   
   definition document {
     relation viewer: user
     permission view = viewer.all(is_satisfied)
   }
   ```

2. Open in Neovim - you should see:
   - `definition` highlighted as keyword
   - `user`, `document` as types
   - `relation`, `permission` as function keywords
   - `.all()` method calls properly highlighted
   - No "Invalid node type" errors

## Grammar Features Supported

- ✅ Import statements
- ✅ Definition and caveat blocks
- ✅ Relations and permissions
- ✅ Binary operators (`in`, `==`, `!=`, etc.)
- ✅ Permission method calls (`.all()`, `.any()`)
- ✅ Arrow operators (`->`)
- ✅ Wildcard types (`*`)
- ✅ Comments and literals

## Troubleshooting

### No Syntax Highlighting
- Verify parser file exists: `~/.local/share/nvim/lazy/nvim-treesitter/parser/authzed.so`
- Verify queries exist: `~/.local/share/nvim/lazy/nvim-treesitter/queries/authzed/highlights.scm`
- Check filetype detection: `:set filetype?` should show `authzed`

### Parser Errors
- Clear cache: `rm ~/.local/share/nvim/lazy/nvim-treesitter/parser-info/authzed*`
- Recompile and reinstall parser
- Restart Neovim

### Grammar Issues
- Test grammar: `tree-sitter parse test.zed`
- Regenerate parser: `tree-sitter generate`
- Check for syntax errors in grammar.js

## Contributing

If you find issues or improvements for this installation process, please open an issue at:
https://github.com/noamsto/tree-sitter-authzed/issues