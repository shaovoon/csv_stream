# C++14 CSV Stream based on C API

### Primary motivation

The purpose is to reduce code bloat in asm.js brought on by use of C++ STL streams. Around 400KB of asm.js can be stave off. 400KB is alot for my 2.8MB application that consisted of SDL2, GLEW, GLM, ZLib and 76 custom classes. 
Usage is similar to [Minimalistic CSV Stream](https://github.com/shaovoon/minicsv). A header-only library like MiniCSV. Just change the namespace from `mini` to `capi`.

### Breaking changes

If you overload the STL stream operators, instead of the CSV stream operators for your custom data type, the class cannot be just a drop-in replacement for MiniCSV. You have to overload the CSV stream operators.

### Optional dependencies

Boost Spirit Qi v2

To use Boost Spirit Qi for string to data conversion, define `USE_BOOST_SPIRIT_QI` before the header inclusion.

```cpp
#define USE_BOOST_SPIRIT_QI
#include "csv_stream.h"
```

To read char as ASCII not integer, define `CHAR_AS_ASCII` before the header inclusion.

```cpp
#define CHAR_AS_ASCII
#include "csv_stream.h"
```


### Benchmark

**Note:** Benchmark results based in latest minicsv v1.8.2. **Note:** Various methods only affects the input stream benchmark.

**File Stream Benchmark**
```
           // minicsv using std::stringstream
           mini::csv::ofstream:  387ms
           mini::csv::ifstream:  386ms
           // minicsv using Boost lexical_cast
           mini::csv::ofstream:  405ms
           mini::csv::ifstream:  283ms
           // capi csv using to_string
           capi::csv::ofstream:  152ms
           capi::csv::ifstream:  279ms
           // capi csv using Boost Spirit Qi
           capi::csv::ofstream:  163ms
           capi::csv::ifstream:  266ms
           // capi in-memory cached file csv
     capi::csv::ocachedfstream:  124ms
     capi::csv::icachedfstream:  127ms
           // capi in-memory cached file csv using Boost Spirit Qi
     capi::csv::ocachedfstream:  122ms
     capi::csv::icachedfstream:  100ms
```

**Note:** In-memory input stream means loading the whole file in memory before processing.

**Note:** In-memory output stream means keeping the contents in memory before saving.

**Caution:** In-memory streams requires sufficient memory to keep file contents on memory.

**String Stream Benchmark**
```
      // minicsv using std::stringstream
      mini::csv::ostringstream:  362ms
      mini::csv::istringstream:  377ms
      // minicsv using Boost lexical_cast
      mini::csv::ostringstream:  383ms
      mini::csv::istringstream:  283ms
      // capi csv
      capi::csv::ostringstream:  113ms
      capi::csv::istringstream:  127ms
      // capi csv using Boost Spirit Qi
      capi::csv::ostringstream:  116ms
      capi::csv::istringstream:  106ms
```

### Caveat

Instantiation is slow because of many data members to initialize.

### Sample code for file stream

```cpp
#include "csv_stream.h"

using namespace capi;

csv::ofstream os("products.txt");
os.set_delimiter(',', "$$");
os.enable_surround_quote_on_str(true, '\"');
if (os.is_open())
{
    os << "Shampoo" << 200 << 15.0f << NEWLINE;
    os << "Towel" << 300 << 6.0f << NEWLINE;
}
os.flush();
os.close();

csv::ifstream is("products.txt");
is.set_delimiter(',', "$$");
is.enable_trim_quote_on_str(true, '\"');

if (is.is_open())
{
    std::string name = "";
    int qty = 0;
    float price = 0.0f;
    while (is.read_line())
    {
        try
        {
            is >> name >> qty >> price;
            // display the read items
            std::cout << name << "," << qty 
                      << "," << price << std::endl;
        }
        catch (std::runtime_error& e)
        {
            std::cerr << e.what() << std::endl;
        }
    }
}
```

### Sample code for cached file stream

```cpp
#include "csv_stream.h"

using namespace capi;

csv::ocachedfstream os;
os.set_delimiter(',', "$$");
os.enable_surround_quote_on_str(true, '\"');
if (os.is_open())
{
    os << "Shampoo" << 200 << 15.0f << NEWLINE;
    os << "Towel" << 300 << 6.0f << NEWLINE;
}
os.write_to_file("products.txt");

csv::icachedfstream is("products.txt");
is.set_delimiter(',', "$$");
is.enable_trim_quote_on_str(true, '\"');

if (is.is_open())
{
    std::string name = "";
    int qty = 0;
    float price = 0.0f;
    while (is.read_line())
    {
        try
        {
            is >> name >> qty >> price;
            // display the read items
            std::cout << name << "," << qty 
                      << "," << price << std::endl;
        }
        catch (std::runtime_error& e)
        {
            std::cerr << e.what() << std::endl;
        }
    }
}
```

### Sample code for string stream

```cpp
#include "csv_stream.h"

using namespace capi;

csv::ostringstream os;
os.set_delimiter(',', "$$");
os.enable_surround_quote_on_str(true, '\"');
if (os.is_open())
{
    os << "Shampoo" << 200 << 15.0f << NEWLINE;
    os << "Towel" << 300 << 6.0f << NEWLINE;
}
os.write_to_file("products.txt");

csv::istringstream is(os.get_text().c_str());
is.set_delimiter(',', "$$");
is.enable_trim_quote_on_str(true, '\"');

if (is.is_open())
{
    std::string name = "";
    int qty = 0;
    float price = 0.0f;
    while (is.read_line())
    {
        try
        {
            is >> name >> qty >> price;
            // display the read items
            std::cout << name << "," << qty 
                      << "," << price << std::endl;
        }
        catch (std::runtime_error& e)
        {
            std::cerr << e.what() << std::endl;
        }
    }
}
```

File Content

```
"Shampoo",200,15.000000
"Towel",300,6.000000
```

Display Output

```
Shampoo,200,15
Towel,300,6
```

### Change delimiter on the fly

Delimiter can be changed on the fly on the input/output stream with `sep` class. The example has whitespace and commma as delimiter in the text.

```cpp
// demo sep class usage
csv::istringstream is("vt 37.8,44.32,75.1");
is.set_delimiter(' ', "$$");
csv::sep space(' ', "<space>");
csv::sep comma(',', "<comma>");
while (is.read_line())
{
    std::string type;
    float r = 0, b = 0, g = 0;
    is >> space >> type >> comma >> r >> b >> g;
    // display the read items
    std::cout << type << "|" << r << "|" << b << "|" << g << std::endl;
}
```

*Version 0.5.2* fixed some char output problems and added NChar (char wrapper) class to write to numeric value [-127..128] to char variables.

```cpp
bool test_nchar(bool enable_quote)
{
    csv::ostringstream os;
    os.set_delimiter(',', "$$");
    os.enable_surround_quote_on_str(enable_quote, '\"');

    os << "Wallet" << 56 << NEWLINE;

    csv::istringstream is(os.get_text().c_str());
    is.set_delimiter(',', "$$");
    is.enable_trim_quote_on_str(enable_quote, '\"');

    while (is.read_line())
    {
        try
        {
            std::string dest_name = "";
            char dest_char = 0;

            is >> dest_name >> csv::NChar(dest_char);

            std::cout << dest_name << ", " << (int)dest_char << std::endl;
        }
        catch (std::runtime_error& e)
        {
            std::cerr << __FUNCTION__ << e.what() << std::endl;
        }
    }
    return true;
}
```

Display Output

```
Wallet, 56
```

[CodeProject Tutorial](https://www.codeproject.com/Articles/1167806/Cplusplus-CSV-Stream-based-on-C-API)
