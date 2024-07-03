---
tags:
  - nvim
  - mnemonic
---

#### Modes

- `i` - Insert
- `a` - Append
- `A` - Big Append, AA, Append After
- `v` - Visual

#### Navigation

`w` - Walk "word"
`W` - Big Walk, Walk "WORD"
`b` - Backward "word"
`B` - Big Backwards, Backward "WORD"
`e` - End "word"
`E` - Big End Word, End "WORD"

#### Commands

 `:s/<old>/<new>` - Substitute "this" for "that".

 `:s/<old>/<new>/g` - Substitute "this" for "that" globally.
	 _(Not happy with that since its not truly global)_

`:r <file_name | !external_command>` - Retrieve content | Read content

`o` - Open Line _(Kind of dumb)_

`O` - Open Line Above, Shift (upper) `o`

`X` - Backspace. X = remove. Standard UI