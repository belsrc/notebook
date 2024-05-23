---
tags:
  - notes
  - nvim
---
## Navigating

| Key         | Action                         |
| ----------- | ------------------------------ |
| `h`         | Move left                      |
| `l`         | Move right                     |
| `k`         | Move up                        |
| `j`         | Move down                      |
| `w`         | Move to start of next word     |
| `e`         | Move to end of next word       |
| `b`         | Move to start of previous word |
| `0`         | Move to start of line          |
| `$`         | Move to end of line            |
| `shift + 0` | Move to start of next line     |

## Exiting

| Key stroke | Action           |
| ---------- | ---------------- |
| `:q`       | Quit operation   |
| `:q!`      | Quit and discard |
| `:wq`      | Quit and save    |

## Editing

Pressing `x` when your cursor is over a character, deletes that character.

Pressing `i` changes to "Insert Mode", typing occurs to the _left_ of current cursor placement.

Pressing `shift + a` changes to "Append Mode", this moves the cursor to the end of the line and allows input.

## Deletion

Pressing `dw` will delete from the current cursor to the next word (delete word), including the space between. This doesn't _have_ to be at the start of the word.

    > This is a [t]est for example sake -> This is a for example sake

But you can also have the cursor mid word to remove the remaining

    > This is a test for ex[a]mple sake -> This is a test for exsake

Pressing `de` will delete from the current cursor to the end of the current word, excluding the space between.

    > This is a test for [e]xample sake -> This is a test for [ ]sake

Pressing `d$` will delete from the current cursor to the end of the line.

    > This is a test[ ]for example sake -> This is a test

Pressing `dd` deletes an entire line.

## Operators & Motions

Many commands are a combination of a operator and a motion. There are several possible motions.

| Motions     | Description                                             |
| ----------- | ------------------------------------------------------- |
| `w`         | To the start of next word, **excluding** first char     |
| `e`         | To the end of next word, **including** the last char    |
| `b`         | To start of previous word, **including** the first char |
| `0`         | To start of line, **including** the first char          |
| `$`         | To the end of line, **including** the last char         |
| `shift + 0` | To start of next line                                   |

Pressing the motion in Normal mode, with no operator, will just move the cursor similarly.

You can add a number before a motion to repeat it _N_ number of times

`2w` will move the cursor twos forward.
`3e` will move the cursor to the end of the third word forward

These can be combined with operators as well.

`d2w` to delete two words forward. Or `2dd` to delete two lines.

Motion format: `operator [number] motion`

### Undo and Redo Commands

`u` to Undo the last operator.
`shift + u` to undo all changes on that line.

`ctrl + r` to redo a command.

### Put Command

`p` operator puts the previously deleted text after the cursor.

`dd` on a line, then move your cursor to the line above where you want it. Press `p` and it will be placed below.

Using `shift + p` will put it **before** your current line.

### Replace Command

`r<char>` to replace the character at the cursor with the one given.

### Change Operator

`ce` change until the end of the word.
This basically is a combination of `de` followed by `i`.

The Change operator uses the same Motions as the Delete operator.

`c [number] motion`

### Cursor Location and File Status

`ctrl - g` (`<C-g>`) will show the file name and position in the file in the footer of the window. May also see the cursor position on the right of the footer if the `ruler` option is turned on.

`[<line number: cursor position]`

`G` to move to the end of the file.

`gg` to move to the start of the file.

You can press `<number> G` to go to the given line number.

### Search Command

<ln: 498>





`zz` to center the current cursor position.

`gx` while cursor is over a link to open in browser

`shift + v` Visual Mode

`:Lazy` opens the Lazy GUI


alacritty
+
tmux
+
nvim
+
kickstart.nvim (merge into the nice things from nvchad)
+
obsidian.nvim



`vim.keymap.set('', '<C-p>, fn, {})`
`C-p` = Ctrl + p




https://www.youtube.com/watch?v=zHTeCSVAFNY'



cssls
eslint
html
jsonls
