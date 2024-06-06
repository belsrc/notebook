(zsh installed by default)

`sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

`vi ~/.zshrc` (edit config as wanted).  Make sure to have
```
[shell]
program = "zsh"
```
Unless you want to set zsh as default for all terminals. May also need to copy paths from `~/bash_profile` over to `~/.zshrc`.

`source ~/.zshrc` (update zsh with any config changes)

`brew install neovim`

`brew install node`
	`node --version` to check version and installation
	`npm --version` to check version and installation

`brew install nvm`

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
	`cargo --version` to check version and installation
	`rustc --version` to check version and installation
	`rustup --version` to check version and installation

`brew install ripgrep`
	`rg --version` to check version and installation

`brew install xclip`
	`echo 123 | xclip`
	`xclip -o` (should print 123)

`brew install fzf`
	`fzf --version` to check version and installation

`npm i -g neovim`

`mkdir ~/.config`
`cd ~/.config`
`git clone https://github.com/nvim-lua/kickstart.nvim.git nvim`

`nvim`
	Should see lazy installing packages
	`:q`
	`:checkhealth` (to check other install aspects)

Install your NerdFont of choice - https://www.nerdfonts.com/

`nvim ~/.config/nvim/init.lua`
In config, change this line to the following: `vim.g.have_nerd_font = true`
This is ~Ln 94 (in the current version)

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

`curl -L https://sw.kovidgoyal.net/kitty/installer.sh | sh /dev/stdin`

https://github.com/GiuseppeCesarano/kitty-theme-OneDark
Copy OneDark.conf to `~/.config/kitty/`
Add `include ./OneDark.conf` to the top of `~/.config/kitty/kitty.conf`






#### Related Configs

- `~/.config/kitty/*`
- `~/.config/nvim/*`
- `~/.zshrc`
- `~/.p10k.zsh`