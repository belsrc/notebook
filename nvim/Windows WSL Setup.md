---
tags:
  - nvim
  - guide
gardening: 🌱
---

Open PowerShell

`wsl --set-default-version 2

`wsl --install -d Ubuntu

`<enter new username and password>

`sudo apt update && sudo apt upgrade

`sudo apt install zsh`

`zsh` (init script stuff. Add and exit)

`sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
	(will ask to make zsh default, pick yes)
	
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

(back in windows) Install a NerdFont
	https://www.nerdfonts.com

`git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k`

`nvim ~/.zshrc`
Change to: `ZSH_THEME="powerlevel10k/powerlevel10k"` save and quit
`source ~/.zshrc`

Run through the prompt.
1. Do you see X icon (3 or 4 ?s) - Input your answer.
2. Do these icons overlap - Your Answer
3. Prompt Style - Rainbow
4. Char Set - Unicode
5. Current Time - 12hr
6. Prompt Separators - Angled
7. Prompt Head - Sharp
8. Prompt Tail - Flat
9. Prompt Height - Two Lines
10. Prompt Connection - Solid
11. Prompt Frame - Full
12. Connection & Frame Color - Lightest
13. Prompt Spacing - Sparse
14. Icons - Many Icons
15. Prompt Flow - Concise
16. Enable Transient Prompt - No
17. Instant Prompt Mode - Verbose
18. Write File - Yes

`git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`

`git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`

`nvim ~/.zshrc`

In config, change this line to the following: `plugins=(git zsh-autosuggestions zsh-syntax-highlighting web-search)`
This is ~Ln 80 (in the current version)

`source ~/.zshrc`

https://apps.microsoft.com/detail/9n0dx20hk701?rtc=1&hl=en-us&gl=US
Install Windows Terminal

Click the down chevron, select "Settings"
- Startup > Default Profile -> Ubuntu
- Defaults > Appearance > Font Face -> Select your Nerd Font
- Open JSON File > https://github.com/yosukes-dev/one-dark-windows-terminal
	- Follow directions







---


`wsl: explorer.exe .` (opens the Linux ~/ folder in Windows)


## Ref Sites

https://github.com/junegunn/fzf

https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH

https://github.com/neovim/neovim/blob/master/INSTALL.md

https://github.com/nvim-lua/kickstart.nvim

https://vimcolorschemes.com/

https://learn.microsoft.com/en-us/windows/wsl/setup/environment#set-up-your-linux-username-and-password


Windows Terminal Config

`%LocalAppData%\Packages\Microsoft.WindowsTerminal_<some hash>\LocalState\settings.json`