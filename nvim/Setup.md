
1. Dank Mono - GOAT
2. Fragment Mono - All ligs. Light, consistent glyphs
3. D2Coding - All ligs. Really light, a little to compact
4. JetBrains Mono - All ligs. Consistent glyphs, a little too heavy
5. Geist - All ligs. Consistent glyphs, a little too heavy




Getting started with Kickstarter (one of nvim's core maintainers)
https://www.youtube.com/watch?v=m8C0Cq9Uv9o



`./font-patcher ~/Downloads/DankMono-491/ttf/DankMono-Regular.ttf`

```
C:\"Program Files (x86)"\FontForgeBuilds\fontforge -script font-patcher P:\codeFonts\DankMono-Italic.otf --name "DankMono Italic Nerd Font" --complete

C:\"Program Files (x86)"\FontForgeBuilds\fontforge -script font-patcher P:\codeFonts\DankMono-Regular.otf --name "DankMono Regular Nerd Font" --complete
```






`:e $MYVIMRC` opens the nvim init.lua



```
~/.config/nvim/init.lua 
~/.config/alacritty/alacritty.toml

%APPDATA%\alacritty\alacritty.toml
```

```toml
# Colors (One Dark)

# Default colors
[colors.primary]
background = '#282c34'
foreground = '#abb2bf'

# Normal colors
[colors.normal]
black   = '#1e2127'
red     = '#e06c75'
green   = '#98c379'
yellow  = '#d19a66'
blue    = '#61afef'
magenta = '#c678dd'
cyan    = '#56b6c2'
white   = '#abb2bf'

# Bright colors
[colors.bright]
black   = '#5c6370'
red     = '#e06c75'
green   = '#98c379'
yellow  = '#d19a66'
blue    = '#61afef'
magenta = '#c678dd'
cyan    = '#56b6c2'
white   = '#ffffff'

[font]
normal = { family = "Dank Mono", style = "Regular" }
size = 16

[window]
dimensions = { columns = 240, lines = 60 }
padding = { x = 12, y = 12 }
```