---
tags:
  - nvim
gardening: 🌿
reference:
  - https://github.com/pinnheads/neovim-tutor
---
## Definitions of Word

This should me noted at the start. There is a difference between a word (usually the lower case command) and WORD (usually the uppercase command).

A "word" is:
- A sequence of letters, digits, underscores, and some special ASCII characters (as specified in the default value of the `iskeyword` option: `@,48–57,_,192–255`).
- Or a sequence of non-blank characters that are not included in the `iskeyword` option.
- Or an empty line.

A "WORD" is:
- A sequence of non-blank characters.
- Or an empty line.

> [!Error] Make Image

```
 w  w    w  w
| || |  | || |
abc@#$  def%^&
|    |  |    |
 WORD    WORD
```

## Navigating

| Key         | Action                         |
| ----------- | ------------------------------ |
| `h`         | Move left                      |
| `l`         | Move right                     |
| `k`         | Move up                        |
| `j`         | Move down                      |
| `w`         | Move to start of next word     |
| `W`         | Move to start of next WORD     |
| `e`         | Move to end of next word       |
| `E`         | Move to end of next WORD       |
| `b`         | Move to start of previous word |
| `B`         | Move to start of previous WORD |
| `0`         | Move to start of line          |
| `$`         | Move to end of line            |
| `shift + 0` | Move to start of next line     |

### Difference In Navigation Commands

> [!Error] Make Images For These

```
`w` movement

Start    w     ww  w w  w w
|        |     ||  | |  | |
function fooBar(obj, obj) {
```

```
`W` movement

Start    W           W    W
|        |           |    |
function fooBar(obj, obj) {
```

```
`b` movement

b        b     bb  b b  b Start
|        |     ||  | |  | |
function fooBar(obj, obj) {
```

```
`B` movement

B        B           B    Start
|        |           |    |
function fooBar(obj, obj) {
```

```
`e` movement

Start  e      ee  ee   ee e
|      |      ||  ||   || |
function fooBar(obj, obj) {
```

```
`E` movement

Start  E           E    E E
|      |           |    | |
function fooBar(obj, obj) {
```


## Exiting

| Key stroke | Action           |
| ---------- | ---------------- |
| `:q`       | Quit operation   |
| `:q!`      | Quit and discard |
| `:wq`      | Quit and save    |

NOTE: Need a Modes table
`i` Insert Mode
`v` Visual Selection Mode (use normal navigation to select text. Can use commands afterwards like `d` to delete the selection)
`a` Insert (Append Before) Mode
`A` Insert (Append After) Mode
`R` Replace Mode


## Editing

Pressing `x` when your cursor is over a character, deletes that character. So it's like backspace but since it removes the character under the cursor, it "moves" towards the right.

Pressing `X` functions like normal backspace. Removing the character to the left of the cursor.

Pressing `i` changes to "Insert Mode", typing occurs to the _left_ of current cursor placement.

Pressing `shift a` changes to "Append Mode", this moves the cursor to the end of the line and allows input.

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
`shift u` to undo all changes on that line.

`ctrl r` to redo a command.

### Put Command

`p` operator puts the previously deleted text after the cursor.

`dd` on a line, then move your cursor to the line above where you want it. Press `p` and it will be placed below.

Using `shift + p` will put it **before** your current line.

### Replace Command

`r<char>` to replace the character at the cursor with the one given.

### Replace Mode

`R` enters into Replace Mode. In this mode each new character typed will replace the character under the cursor and advance the cursor by one. Hitting `esc` exits the mode.

### Change Operator

`ce` change until the end of the word.
This basically is a combination of `de` followed by `i`.

The Change operator uses the same Motions as the Delete operator.

`c [number] motion`

### Cursor Location and File Status

`ctrl g` (`<C-g>`) will show the file name and position in the file in the footer of the window. May also see the cursor position on the right of the footer if the `ruler` option is turned on.

`[<line number: cursor position]`

`G` to move to the end of the file.

`gg` to move to the start of the file.

You can press `<number> G` to go to the given line number.

### Search Command

`/<phrase>` to search for the entered text.
If there is multiple occurrences:
`n` to go to next.
`N` to go to previous.

This searches the document top down.

To search the document in the reverse order use `?<phrase>` instead.

Use `ctrl - o` (`<C-o>`) to go to the older position. And `ctrl i` (`<C-i>`) to go to the newer position. 


### Matching Parentheses

Place the cursor on an opening `(`, `[` or `{` and press `%`. The cursor will move to the matching closing symbol. Depending on your setup, the matching symbols will also be highlighted.


### Substitute Command

`:s/<old>/<new>` to replace the first occurrence of `old` with `new` for the current line.

Using `:s/<old>/<new>/g` will replace all occurrences on the current line.

The changes won't be committed until you hit `enter`.

You can use `:#,#s/<old>/<new>/g` to replace all occurrences from the first line number (`#`) to the second line number, inclusively.

`:%s/<old>/<new>/g` to change every occurrence in the entire file.

`:%s/<old>/<new>/gc` to find every occurrence in the entire file with a prompt whether to replace it or not.

The `c` (confirm) flag can be added to all of the other substitutes command as well.


### External Commands

`:!<command>` this allows you to execute any external shell command.

As an example `:!ls -la`.


### Writing (Saving) Files

To save changes in a new file use:

`:w <file-name>`

You can also save just a portion of the current file.

Press `v` to enter Visual Character Mode. Then you `h, j, k, l` to select the text that you want to save. After that press `:`. You will see `:'<,'>` appear at the bottom of the screen.

Now type the same `:w <file-name>` and press enter.


## Retrieving File Contents

You can retrieve the contents of a file and place them _below_ the cursor using:

`:r <file_name>`

You can also read the output of an external command.

`:r !<command>`


## Open Command

Pressing `o` will add a new line below the current cursor line and put you into Insert mode. This is similar to doing `A` followed by `return`.

To open a line ABOVE your cursor use `O`


### Copy and Paste

Enter Visual Mode by typing `V`, then you can use the standard navigation to select text. Use the `y` operator (Yank) to copy the selected text.

Then you can use `p` to the put the text after the cursor.

`y` can also be used as an operator. i.e. `yw` will copy a word.

You can use `P` to put before the cursor.


### Set Options

`:set ic` to "Ignore Case" when searching

`:set hls` (hlsearch) to "highlight search" which highlights all matches

`:set is` (incsearch)  to highlight partial matches

Can turn them off using `no__`

`:set noic`

You can invert the current using `inv__`

`:set invic`


### Help

You can use `F1` or `:help` to open the help window. You can use `ctrl w, ctrl w` to switch between the two windows.


### Completion

You can use `ctrl d` and `tab` for command autocompletion. 

For example, run `:!ls` to list files. Now `:e` and then `ctrl d` to see a list of commands starting with `e`. You can use `tab` to cycle through the available commands.


### References

`:Tutor`






`zz` to center the current cursor position.

`gx` Open file path or URI under cursor

`gf` Go to file under cursor

`:Lazy` opens the Lazy GUI

`\` to open sidebar file explorer

`<leader>sf` - Search Files

`<leader>s/` - Search Buffer

`<leader>ds` - Document Symbols (functions, etc)

`~` - toggle case

`C-v` or `C-q` (on win) - Vertical Selection

`c` - change. `cb` - change back word, `cw` - change word



`vim.keymap.set('', '<C-p>, fn, {})`
`C-p` = Ctrl + p




https://www.youtube.com/watch?v=zHTeCSVAFNY'


