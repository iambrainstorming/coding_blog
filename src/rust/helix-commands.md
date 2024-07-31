# Helix commands

## Find and replace

Press % to select all the text

Then s and type the text you want to select and replace the text
Then c to replace it.

Try the example:

Replace the python python python python

## Collapsing the selection

 Type ; to collapse selections to single cursors.

 Sometimes, you want to deselect without having to move the
 cursor(s). This can be done using the ; key.

 1. Move the cursor to the line marked '-->' below.
 2. Use the motions you have learned to move around the line,
    and try using ; to deselect the text after it is selected
    by the motions.

 --> This is an error-free line with words to move around in.

 Note: This works the same in Select mode.
 Note: Another related command is Alt-; which flips selections.

## words and WORDS

Some common motions include:
   w - Move forward to before the beginning of the next word.
   e - Move forward to the end of the current word.
   b - Move backward to the beginning of the current word.

## Change command

 Type c to change the current selection.

 The change command deletes the current selection and enters
 Insert mode, so it is a very common shorthand for di.

## Yank and paste
Type y to yank (copy) the selection.

Type p to paste the yanked selection after the cursor.

Type P to paste the yanked text before the cursor.

## Searching a file

Type / to search forward in file, Enter to confirm search.

Type n to go to the next search match.

Type N to go to the previous search match.

## Multiple cursor

Type capital C to duplicate the cursor to the next suitable line.
Type , to remove the cursor. 

## Align selections

Type & to align the contents of the selections.

 1. Move the cursor to the first line marked '-->' below. Place
    the cursor on the whitespace just after the arrow.
 2. Type C four times or 4C.
 3. Type W to select the numbers and brackets.
 4. Type & to align the words.

```
--> 97) lorem
--> 98) ipsum
--> 99) dolor
--> 100) sit
--> 101) amet
```
 Note: & only cares about the alignment of the "head" of the
       selections - the end that moves. The other end is called
       the "anchor".

## Replace with yanked text

Type R to replace the selection with previously yanked text.

Space + R to replace selections by clipboard contents

 1. Move the cursor to the line marked '-->' below.
 2. Type w to select "watermelons" and then y to yank it.
 3. Select "oranges" with w.
 4. Type R to replace "oranges" with "watermelons"


 --> I like watermelons because oranges are refreshing.
     I like watermelons because watermelons are refreshing.

## Joining lines together

Type J to join together lines in selection.

Select lines and type J

## Changing case

Type ~ to switch the case of all selected letters.

Type \` to set all selected letters to lowercase.

Press Alt-\` to set all selected letters to uppercase.

## Goto line number

Enter to command mode with :

type :g 100

100 is the line number

Ctrl-u,	Move cursor and page half page up,	page_cursor_half_up

Ctrl-d,	Move cursor and page half page down,	page_cursor_half_down

## Goto

Press g in Normal mode

g, start of the file

e,	Go to the end of the file,	goto_last_line

d,	Go to definition (LSP),	goto_definition

y,	Go to type definition (LSP),	goto_type_definition

r,	Go to references (LSP),	goto_reference

i,	Go to implementation (LSP),	goto_implementation


## Window mode

Accessed by typing Ctrl-w in normal mode.


w, Ctrl-w, Switch to next window,	rotate_view

v, Ctrl-v,	Vertical right split,	vsplit

s, Ctrl-s,	Horizontal bottom split,	hsplit





