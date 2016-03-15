Autocompletion in `main()`
--------------------------

IDE opened the file correctly, and parsed it without error.

Opening up autocompletion with `Ctrl+Space` triggers the following pop-up dialog:

![no-type](_1.png)

Many names are listed, but there's no visible distinction which names are types, functions or variables.

![std_](_2.png)

Only after typing `::` Code::Blocks does more intelligent suggestions.

![std_m](_3.png)



![std_misma](_4.png)

Relevant functions are suggested, but no info about arguments until the function is selected with `Tab`.

![std_imsm](_5.png)

Code::Blocks doesn't do fuzz-matching.