---
tags:
  - notes
  - nvim
---
## Movement

| Key | Action     |
| --- | ---------- |
| `h` | Move left  |
| `l` | Move right |
| `k` | Move up    |
| `j` | Move down  |

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

Pressing `dw` will delete from the current cursor to the next word (delete word), including the space between. This doesn't _have_ to be at the start of the word.

    > This is a [t]est for example sake -> This is a for example sake

But you can also have the cursor mid word to remove the remaining

    > This is a test for ex[a]mple sake -> This is a test for exsake

Pressing `de` will delete from the current cursor to the end of the current word, excluding the space between.

    > This is a test for [e]xample sake -> This is a test for [ ]sake

Pressing `d$` will delete from the current cursor to the end of the line.

    > This is a test[ ]for example sake -> This is a test
    

## Operators & Motions

Many commands are a combination of a operator and a motion. For example the delete operator `d`.  `d <motion>`

Typing a number before a motion repeats it _n_ times.







`w` moves to start of next word
`e` moves to end of next word