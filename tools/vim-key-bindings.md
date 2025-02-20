# Vim Key Bindings – Vim Keys List Reference

## Table of Content
- [Vim Modes](#vim-modes)
- [How to Navigate using Vim Key Bindings](#how-to-navigate-using-vim-key-bindings)
- [How to Edit Text in Normal Mode](#how-to-edit-text-in-normal-mode)
- [Insert Mode](#insert-mode)
- [Visual Mode](#visual-mode)
- [More Navigation Commands](#more-navigation-commands)
- [Programming Specific Key Bindings](#programming-specific-key-bindings)
- [Conclusion](#conclusion)

## Vim Modes
Vim operates in distinct modes, each designed for specific tasks:
- Normal Mode: Default state for efficient movement, deletion, searching, and more.
- Insert Mode: Allows you to type text directly into your file.
- Visual Mode: Enables text selection for manipulation.

## How to Navigate using Vim Key Bindings

### Basic Movement
Here are some basic movement keys:

```
h  --> move left
j  --> move down
k  --> move up
l  --> move right
```

### Moving to Top and Bottom
You can use these keys to move to the top or bottom of the page:

```
gg  --> Move to the first line of the page
G   --> Move to the last line of the page
```

### Moving to a Specific Line
You can move to a specific line using : followed by the line number:

```
:number   --> Go to the line number
```

### Moving Within a Line
You can use these keys move to the start or end of a line:

```
$  --> Go to the end of the line
0  --> Go to the start of the line
```

### Moving Through Words
Here are some keys for moving through lines of text:

```
b  --> Go to the previous word
w  --> Go to the next word
```

## How to Edit Text in Normal Mode

### Deletion and Copying
You can delete or copy characters using these keys:

```
x   --> Delete the character under the cursor
dd  --> Delete the entire line
yy  --> Copy (yank) the entire line
p   --> Paste the previously deleted or copied text after the cursor
```

### Undo and Redo
```
u          --> Undo the last action
Ctrl + r   --> Redo the undone action
```

### Searching and Replacing
```
/              --> Start searching forward
?              --> Start searching backward
:s/old/new/g   --> Replace all occurrences of "old" with "new" in the entire file
```

## Insert Mode
In Insert mode, you can type text directly into your file. Here are some key bindings to help you navigate in this mode:

```
Esc   --> Return to Normal Mode
i     --> Start inserting text before the cursor
a     --> Start inserting text after the cursor
o     --> Open a new line below the current line and start inserting text
```

## Visual Mode
Visual mode is useful for selecting and manipulating text visually. Here are some key bindings:

```
v          --> Start character-wise visual mode
V          --> Start line-wise visual mode
Ctrl + v   --> Start block-wise visual mode
d          --> Delete the selected text
y          --> Copy (yank) the selected text
p          --> Paste the copied text after the cursor
```

## More Navigation Commands

### Scrolling
```
Ctrl + u   --> Move half a screen up
Ctrl + d   --> Move half a screen down
Ctrl + b   --> Move one full screen up
Ctrl + f   --> Move one full screen down
```

### Jumping between Words and Paragraphs
```
(  --> Jump to the beginning of the previous sentence
)  --> Jump to the beginning of the next sentence
{  --> Jump to the beginning of the previous paragraph
}  --> Jump to the beginning of the next paragraph
```

## Programming Specific Key Bindings

### Moving Between Functions
In a code file, navigating between functions is a common task. Vim makes it efficient with these keys:

```
]]  --> Move to the beginning of the next function
[[  --> Move to the beginning of the previous function
```

### Indentation
Maintaining consistent code indentation is crucial. Vim enhances this with these keys:

```
>>  --> Indent the current line to the right
<<  --> Indent the current line to the left
```

### Folding
Code folding aids in managing large files. Vim provides powerful folding commands:

```
zf{motion}  --> Create a fold (replace {motion} with a movement command)
zo          --> Open a fold
zc          --> Close a fold
zr          --> Reduce folding level throughout the file
zm          --> Increase folding level throughout the file
```

### Code Commenting
Efficiently comment and uncomment code with:

```
gcc  --> Comment/uncomment the current line
gc{motion}  --> Comment/uncomment the lines covered by {motion}
```

### Matching Parentheses
Effortlessly navigate and understand code structure with:

```
%  --> Move to the matching parenthesis, bracket, or brace
```

## Conclusion
In essence, mastering Vim key bindings unlocks a world of efficient text editing. From basic movements to advanced coding tasks, Vim's modes and commands streamline navigation and manipulation.

By embracing these key bindings, you'll enhance productivity and enjoy a more seamless editing experience. Happy coding in Vim!
