Autocompletion in `main()`
--------------------------

IDE opened the file correctly, and parsed it without error.

Opening up autocompletion with `Ctrl+Space` triggers the following pop-up dialog:

![no-type](_1.png)

Listed are names from standard library, and from current file. Note that IDE gave up on constants with more complicated types: `przedmioty_init` with type `const std::array<std::string, 3>` is listed as `const ?`.

![std_](_2.png)

Listed are some global variables, and some irrelevant functions that are implementation details.

![std_m](_3.png)

Thankfully it gets slightly more helpful after the first letter.

![std_misma](_4.png)

Relevant functions are suggested with full argument types and names. 

![std_imsm](_5.png)

Eclipse CDT doesn't do fuzz-matching.