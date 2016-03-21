Hello World
-----------


`main.cpp`

```
#include <iostream>

int main(int argc, char **argv) {
    std::cout << "Hello, world!" << std::endl;
    return 0;
}
```

- no `using namespace std;` 
- uses `std::endl`
- has commandline arguments although they're unused
- has `return 0;` at the end