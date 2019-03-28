maybe
=====

A really simple return value based error checking library. Focused on no memory allocations and no exceptions for extreme environments where these are hard or not trivial to do.

Example
-------
```cpp
#include <iostream>
#include "zpp/maybe.h"

namespace my_namespace
{

// Use arbitrary error enumeration whose underlying type is int.
enum class my_error : int
{
    success = 0,
    something_bad = 1,
    something_really_bad = 2,
};

// Define this function which returns the error descriptions and
// the error category name.
const zpp::error_category & category(my_error)
{
    return zpp::make_error_category<my_error>("my_category",
        [](auto code) -> std::string_view {
            switch (code) {
                case my_error::success:
                    return {};
                case my_error::something_bad:
                    return "Something bad happened.";
                case my_error::something_really_bad:
                    return "Something really bad happened.";
                default:
                    return "Unknown error occurred.";
            }
        }
    );
}

// Example of function that supposed to return an integer but
// fails sometimes with an error.
zpp::maybe<int> foo(bool value)
{
    if (!value) {
        // Fail with error code.
        return my_error::something_bad;
    }

    // Success
    return 1337;
}

// This function is going to call a function that fails and
// check the error.
void bar()
{
    // Change this to observe a different behavior.
    bool is_success = false;

    if (auto result = foo(is_success)) {
        // Success path.
        std::cout << "Success path: value is '" << result.value() << "'\n";
    } else {
        // Fail path.
        std::cout << "Error path: error is '" << result.error().code()
            << "', '" << result.error().message() << "'\n";
    }
}
} // my_namespace
```