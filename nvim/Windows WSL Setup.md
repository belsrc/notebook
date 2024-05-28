---
tags:
  - nvim
  - guide
---

Open PowerShell

`wsl --set-default-version 2

`wsl --install -d Ubuntu

`<enter new username and password>

`sudo apt update && sudo apt upgrade

`sudo apt install zsh`

`zsh` (init script stuff. Add and exit)

`sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
	(will ask to make zsh, pick yes)
	
`echo $SHELL` (should output "/bin/zsh")

`vi ~/.zshrc` (edit config as wanted)

`source ~/.zshrc` (update zsh with any config changes)

`sudo add-apt-repository ppa:neovim-ppa/unstable -y`
`sudo apt-get install neovim`

`sudo apt install make`
	`make -version` to check version and installation

`sudo apt install unzip`
	`unzip -v` to check version and installation

`sudo apt install gcc`
	`gcc --version` to check version and installation

`sudo apt install npm nodejs`
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash`
`nvm install node`
	`node --version` to check version and installation
	`npm --version` to check version and installation

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
`source $HOME/.cargo/env`
	`cargo --version` to check version and installation
	`rustc --version` to check version and installation
	`rustup --version` to check version and installation

`sudo apt-get install ripgrep`
	`rg --version` to check version and installation

`sudo apt-get install xclip`
	`echo 123 | xclip`
	`xclip -o` (should print 123)

`git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf`
`~/.fzf/install`
`source ~/.zshrc`

`npm i -g neovim`

`mkdir ~/.config`
`cd ~/.config`
`git clone https://github.com/nvim-lua/kickstart.nvim.git nvim`

`nvim`
	Should see lazy installing packages
	`:q`
	`:checkhealth` (to check other install aspects)

(back in windows) Install alacritty
	https://alacritty.org

Install a NerdFont
	https://www.nerdfonts.com

Edit `%APPDATA%\alacritty\alacritty.toml`

```toml
[shell]
program = "Ubuntu"

[window]
dimensions = { columns = 240, lines = 60 }
padding = { x = 12, y = 12 }
opacity = 0.98
blur = true

[font]
# Name of installed NerdFont
normal = { family = "Dank Mono Nerd Font", style = "Regular" }
size = 12

# Colors (One Dark)
# Or whatever theme you want
[colors.primary]
background = '#282c34'
foreground = '#abb2bf'

[colors.normal]
black   = '#1e2127'
red     = '#e06c75'
green   = '#98c379'
yellow  = '#d19a66'
blue    = '#61afef'
magenta = '#c678dd'
cyan    = '#56b6c2'
white   = '#abb2bf'

[colors.bright]
black   = '#5c6370'
red     = '#e06c75'
green   = '#98c379'
yellow  = '#d19a66'
blue    = '#61afef'
magenta = '#c678dd'
cyan    = '#56b6c2'
white   = '#ffffff'
```

Next time you open Alacritty, it will use the WSL.


`wsl: explorer.exe .` (opens the Linux ~/ folder in Windows)


## Ref Sites

https://github.com/junegunn/fzf

https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH

https://github.com/neovim/neovim/blob/master/INSTALL.md

https://github.com/nvim-lua/kickstart.nvim

https://vimcolorschemes.com/

https://learn.microsoft.com/en-us/windows/wsl/setup/environment#set-up-your-linux-username-and-password