

VI CLONES

stevie
elvis
vile - vi like emacs
vim
neovim
nvi
busybox/vi - minimal set of vi features
gvim




START FLAGS

-o - start with all windows in horizontal split
-O - start with all windows in vertical split


VIMRC

:help vimrc
" - comment in vimrc




OPTIONS

:set tabstop=4
:set nowrap
:set splitbelow - split a new window below the current window
:set splitright - split a new window right of current window
:set sidescroll=10
:set linebreak - wrap on word breaks rather than in between words
:set list - show end of line characters (\$)
:set expandtab ts=2 sw=2 ft=yaml syntax=yaml


MODES

normal/command
insert
execute
visual




NORMAL/COMMAND MODE

repeat last command - .
undo last change - u
redo last change - ctrl+r

move cursor - hjkl
go to insert mode - i
go to high, medium and low of screen - H M L

scroll:
scroll viewport up a line - ctrl-y
scroll viewport down a single line - ctrl-e
scroll up half a page - ctrl-u
scroll down half a page - ctrl-d

search:
forward search - /
backward search - ?


insert/append/replace:
go to insert mode at current character - i
go to insert mode after current character - a
Insert at beginning of line, capital i - I
Append at end of line - A \[text to append\]
go to insert mode after current line - o
go to insert mode before current line - O \[*text to insert*\]
replace at current character - r
go to replace mode - R


delete:
delete character under cursor - x
delete character before cursor - shift+x
d\^ - delete all white space till first character
shiff+d - delete till end of line

cut :
dd - delete a single line
cc OR shift+c - delete a single line and go to insert mode


copy:
yy


paste:
p
shift+p


change case:
\~ - invert case of current character
g\~ - invert case of current character
gu
gU
gUU - change case of entire line to upper case
guu - change case of entire line to lower case
g\~\~ - invert case of the current line

indent:
\<\<
\>\>


save:
shift+zz - like :wq of execute mode

TBS:
increment first number in line - ctrl+a
decrement first number in line - ctrl+x



NOTE: you can use a number before a command and the number will be
interpreted in the context of the comand, e.g. 5\>\> wil indent 5 lines
and 5dd will cut 5 lines




MOTIONS OF NORMAL/COMMAND MODE

line:
0 - start of line (zero)
\^ - start of line
\$ - end of line

word:
w, W - next word
b, B - previous word
e, E - end of word

char in line:
f\[char\] - forward to char (inclusive)
F\[char\] - backward to char (inclusive)
t\[char\] - forward to char (exclusive)
T\[char\] - backward to char (exclusive)
; - next match
, - previous match

text object:
{ - beginning of paragraph
} - end of paragraph
) - next sentence
( - previous sentence
% - matching pair of ( ) \[ \] { }

go to:
gg - go to beginning of document
G - go to end of document
\<line_number\>G - go to line number
ge - backwards to end of word
gE - backwards to end of WORD
\`. - go to last insert location
\`" - go to last exit buffer



COMBINE COMMANDS WITH MOTIONS IN NORMAL/COMMAND MODE

Note: The command comes first and the motion comes next

delete:
dw - delete word (forward, delimiters are non word characters like
space, colon, comma, etc)
d\$ - delete till end of line
d0 or d\^ - delete till start of line
df\[char\] - delete till char
dt\[char\] - delete till char

change:
cw - change word

yank
yw - yank word


change case:
g\~w
guw
gUW






EXECUTE MODE

help
:help \[\<topic\>\] - e.g. :help ctrl-w, :help arg
:help \[*area*\]\[*topic*\] -
i - insert mode e.g. :help i_ctrl-w
v - visual mode e.g. :help v_ctrl-w
: - ex mode e.g. :help :q, :help :delete
-*flag* - help on startup flag e.g. :help -o

:\[text\] ctrl-d - see options for text


options
:set all - see all possible options for vim
:options - all options with description
:set \[option\]? - get value of option
:set \[option\]! - invert a boolean option
:set \[option\] \[option\] \[option\] \... - set one or more options



shell commands
:! seq 9 - :! runs a command on the command line outside vim - in this
case seq 9
:read ! seq 9 - :read ! takes the output of a command and puts it into
the vim file, in this case seq 9
:.,\$!grep 9 - will send the contents of the open file to the command
'grep 9' and return the output back into the file



file explorer

Note: the builtin file explorer is called netrw

:E\[xplore\] - file explorer
:Sexplore - split explorer horizontally
:Vexplore - split explore vertically

:help netrw
:help netrw-quickmap - get help on netrw
I did not write the quick commands here, go to help and read them!


ranges
.,.+5 - current line to current line + 5 lines
5,11 - line numbers 5 to 11 inclusive
0,\$ - beginning and end of document
.,/regex/ - from current line to whichever first line has the regex
/regex/,/regex/ -
/regex/+\[num\],/regex/-\[num\] - e.g. /alpha/+1,/gama/-1 will give a
range of the lines between the lines that have alpha and gama
% - the currently open buffer, meaning the whole document


editing
:e \[*file name*\] - edit file
:\[range\] delete (d)
:\[range\] yank (y)
:\[range\] copy
:\[range\] move \[line number\] - :4,5 move 0 (will move lines 4 to 5 to
top of file)
:put - will put after the line the cursor is on or the one specfied in
range
:\[range\] substitute (s) /pattern1/pattern2/\[g\]


wrting and saving
:w \[*file name*\]
:file \[*file name*\] - specify the file name being worked on in this
buffer

:saveas \[*file name*\]
:write \>\> \[*file name*\] - append to file
:e! - remove unsaved changes in the current buffer
:qall - quit all
:wall - write all
:wqall


Buffers:
:buffers
:buffer \[*inded number or file name*\]* -* go to buffer with file name
:ls - list all open files (buffers)
:bn or :n - go to next buffer
:bp or p - go to previous buffer
:b \[*index number*\]
:bd \[*index number or file name*\] - buffer delete
:next - go to the next open file
:previous - go to the previous open file


Splits:
:split \[*file*\] or ctrl+w,s - split horizontal

:vsplit or ctrl+w,v - split vertical

:new - horizontal split with new file
:vnew - vertical split with new file
:\[*num lines*\] ctrl+w, \[+\|-\] - resize
ctrl+w,w \| \<arrow key\> - toggle between splits

:sb\[n\] - split buffer number n horizontally
:vert sb\[n\] - split buffer number n vertically

:ba - split all buffers horizontally
:vert ba - split all buffers vertically

:Vexplore - vertical split and open explorer
:Sexplore - horizontal split and open explorer


Tabs:
:tabnew - create new tab
:tabclose


Windows:
:close - close the active window
:only - close all other panes


Normal Mode commands:
:normal \[normal mode command\] - execute normal mode commands, e.g.
:normal dd



TBS

:vimgrep

:copen

:bufdo!

:abbr NSA no such agency - will replace NSA with 'no such agency'

:\<line number\> - go to line number

:set list - see line endings



INSERT MODE


delete:
delete word before cursor - ctrl+w
delete line before cursor - ctrl+u


go to:
shift+right arrow - go to next word
shift+left arrow - go to previous word


TBS:
execute one normal mode command - ctrl+o
keyword completion base on next match - ctrl+n





VISUAL MODE


Visual Mode
v


Visual Block:
ctrl + v
arrow keys to select block on which you want to work
if you want to add:
shift + i for insert mode
insert the character you want
press esc
this will apply to all lines in block that were selected
if you want to delete:
press d to delete
esc



Visual Line:
shift + v




MACRO

start recording a macro
q
record the register of the macro (any alphanumeric key)
h
press any command sequence, this will become the macro
<command sequence>
stop recording the macro
q
replay the macro by recalling the register with @
@h




SOLUTIONS IN VIM

I want to select all text in a file
v
gg
G
:


I want to sort a file
v
gg
G
:
sort
