Autocompletion in `main()`
--------------------------

IDE opened the file correctly, and parsed it without error.

Opening up autocompletion with `Ctrl+Space` triggers the following pop-up dialog:

![no-type](_1.png)

Listed are names from current file, and included in the headers. Also "quick snippets" and preprocessor directives.

![std_](_2.png)



![std_m](_3.png)



![std_misma](_4.png)

Only function names are displayed in the listbox. Multiple overloads are congregated into one selection. 

![std_imsm](_5.png)

Visual Studio trying to do fuzz-matching.  

![std_ism](_6.png)

`std::mismatch` is matched even when the first letter is missing.