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
- `R` - Replace

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

`f<symbol>` = find first symbol

`%` = find matching symbol

`f{v%d` = `f`ind next `{`, enter `v`isual mode, select to matching `}` (inclusive) and then `d`elete

Can use `F` to find backwards

`va{` = `v`isual `a`round `{`, select the same as above.

`va{V` = select the body (above) and the definition (rest of fn), complete fn delete

`vaw` = `v`isual `a`round `w`ord, select just word

`vi'` = `v`isual `i`nside `'`. Can be any symbol.