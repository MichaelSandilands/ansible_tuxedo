# apt Role

This role is responsible for installing common development tools and utilities on the target system using the `apt` package manager.

## Purpose

The primary goal of this role is to provision the machine with essential software packages required for general development tasks, ensuring a consistent and ready-to-use environment.

## Included Packages

This role installs the following packages, defined in `vars/main.yml`:

* `build-essential`: Provides essential build tools like `gcc`, `g++`, and `make`.
* `curl`: A command-line tool for transferring data with URLs.
* `fd-find`: A fast and user-friendly alternative to `find`.
* `fonts-noto-color-emoji`: Noto Color Emoji font for full-color emojis.
* `fzf`: A command-line fuzzy finder.
* `git`: The popular version control system.
* `imagemagick`: Software suite to create, edit, compose, or convert bitmap images.
* `libmagickwand-dev`: Development files for ImageMagick.
* `lua5.1`, `liblua5.1-0-dev`, `luajit`: Lua programming language and its JIT compiler, along with development files.
* `make`: GNU Make utility to maintain groups of programs.
* `ripgrep`: A line-oriented search tool that recursively searches the current directory for a regex pattern.
* `tree-sitter-cli`: Command-line interface for Tree-sitter parsers.
* `tmux`: A terminal multiplexer.
* `unzip`: Utility for extracting zipped archives.
* `wget`: Non-interactive network downloader.
* `xclip`: Command-line interface to X selections (clipboard).

## Usage

To use this role, include it in your main playbook (`local.yml` in this project) under the `roles` section:

```yaml
# local.yml
# ...
  roles:
    # ... other roles
    - role: apt
# ...
```
