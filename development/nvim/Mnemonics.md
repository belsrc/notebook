---
tags:
  - nvim
  - mnemonic
---
## Modes

- `i` : Insert
- `I` : Insert Start Line
- `a` : Append
- `A` : Append End Line
- `v` : Visual
- `V` : Visual Line
- `R` : Replace

(`C-v` or `C-q` (on win) - Vertical Selection)

## Navigation

- `w` : Walk "word"
- `W` : Big Walk, Walk "WORD"
- `b` : Backwards "word"
- `B` : Big Backwards, Backward "WORD"
- `e` : End "word"
- `E` : Big End, End "WORD"

## Visual `v`

`va{` : `v`isual `a`round `{`, select the same as above.

`viw` : `v`isual `i`nside `w`ord, select just word

`vi'` : `v`isual `i`nside `'`. If not currently in between `''`, will find next set. Can be any symbol.

`va{V` : select the body (above) and the definition (rest of fn), complete fn delete

## Change `c`

`cb` : change back word

`cw` : change word

`ciw` : change inside word (deletes word and enters insert). 

`ci{` : change inside next `{}`. If not currently in between `{}`, will find next set.

`caw` : change around word

`ca{` : change around next `{}` . If not currently in between `{}`, will find next set.

`cf(` : change to next found `(`.

`cap` : change around paragraph. i.e. until next blank line.

Number motions can also be used:

`2ci'` : change inside second pair of `''`.

## Delete `d`

`db` : delete back word

`dw` : delete word

`diw` : delete inside word (deletes word and enters insert). 

`di{` : delete inside next `{}`. If not currently in between `{}`, will find next set.

`daw` : delete around word

`da{` : delete around next `{}` . If not currently in between `{}`, will find next set.

`df(` : delete to next found `(`.

`dap` : delete around paragraph. i.e. until next blank line.

Number motions can also be used:

`2di'` : delete inside second pair of `''`.

## Goto `g`

`gx` : goto Execute (file path or URI)

`gf` : goto file under cursor

`gd` : goto definition

## Find `f`

`f<symbol>` : find first symbol

Can use `F` to find backwards

`f{v%d` : find next `{`, enter `v`isual mode, select to matching `}` (inclusive) and then `d`elete

## Uncategorized

`o` : Open Line _(Kind of dumb)_

`O` : Open Line Above, Shift (upper) `o`

`X` : Backspace. X = remove. Standard UI

`%` : find matching symbol

`zz` : to center the current cursor position.

`~` : toggle case

## Commands

 `:s/<old>/<new>` : Substitute "this" for "that".

 `:s/<old>/<new>/g` : Substitute "this" for "that" globally.
	 _(Not happy with that since its not truly global)_

`:r <file_name | !external_command>` : Retrieve content | Read content
