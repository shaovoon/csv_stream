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
// For version 0.5.3 and above, give empty string 
// for the escape string(2nd parameter).
// Text with comma delimiter will be 
// enclosed with quotes to be
// compatible with MS Excel CSV format.
os.set_delimiter(',', "");
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
// For version 0.5.3 and above, give empty string 
// for the escape string(2nd parameter).
// Text with comma delimiter will be 
// enclosed with quotes to be
// compatible with MS Excel CSV format.
os.set_delimiter(',', "");
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
// For version 0.5.3 and above, give empty string 
// for the escape string(2nd parameter).
// Text with comma delimiter will be 
// enclosed with quotes to be
// compatible with MS Excel CSV format.
os.set_delimiter(',', "");
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

__Version 0.5.2__ fixed some char output problems and added NChar (char wrapper) class to write to numeric value [-127..128] to char variables.

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

## Available public member function interface

### Public member functions of istream_base inherited by ifstream and istringstream

```cpp
// Set newline unescaped text. Default is "&newline;"
void set_newline_unescape(std::string const& newline_unescape_);

// Get newline unescaped text.
std::string const& get_newline_unescape() const;

// Set delimiter and its unescaped text, meaning the unescaped text shall be
// replaced with the delimiter if unescaped text is encountered in the input.
void set_delimiter(char delimiter_, std::string const& unescape_str_);

// Get delimiter 
std::string const& get_delimiter() const;

// Get the unescaped text of delimiter
std::string const& get_unescape_str() const;

// Get the current delimited text
const std::string& get_delimited_str();

// Enable trimming on the string input. unescape shall be replaced with quote
// when encountered in the input. Default unescaped text is "&quot;"
void enable_trim_quote_on_str(bool enable, char quote, const std::string& unescape);

// Get the rest of line that still haven't been delimited yet.
std::string get_rest_of_line() const;

// Enable blank line in between. 
void enable_blank_line(bool enable);

// If enabled, the parsing shall terminate
// when blank line is encountered.
void enable_terminate_on_blank_line(bool enable);

// Query if terminate_on_blank_line is enabled
bool is_terminate_on_blank_line() const;

// Returns number of delimiter in the current line.
// Prefers to call after readline()
size_t num_of_delimiter() const;

// Get the original unparsed line
const std::string& get_line() const;
```

#### Public member functions of ifstream and icachedfstream (File stream for reading)

```cpp
// Open a text file for reading
void open(const std::string& file);
void open(const std::wstring& file);
void open(const char * file);
void open(const wchar_t * file);

// Query whether the file is opened successfully.
bool is_open();

// Reset all the member variables
void reset();

// Close the file.
void close();

// Skip this line. Used when the line does not contain delimiter you want.
void skip_line();

// Read the next line. Must be called before the << operator is called.
bool read_line();
```

#### Public member functions of istringstream (String stream for reading)

```cpp
// Set new input string for processing.
void set_new_input_string(const std::string& text);

// Reset all the member variables
void reset();

// Skip this line. Used when the line does not contain delimiter you want.
void skip_line();

// Read the next line. Must be called before the << operator is called.
bool read_line();
```

### Public member functions of ostream_base inherited by ofstream and ostringstream

```cpp
// Enable surround the string input with quote. When quote is encountered in 
// the input, it is replaced with escape. Default escaped text is "&quot;".
void enable_surround_quote_on_str(bool enable, char quote, const std::string& escape);

// Set newline escaped text, meaning newline encountered in the output shall
// be replaced with the escape text. Default escape text is "&newline;"
void set_newline_escape(std::string const& newline_escape_);

// Get newline escaped text.
std::string const& get_newline_escape() const;

// Set delimiter and its escaped text, meaning the delimiter shall be
// replaced with this unescaped text if delimiter is encountered in the output.
// For version 1.8.5 and above, give empty string for the escape string(2nd parameter) for
// text with comma delimiter will be enclosed with quotes to be compatible with MS Excel CSV format.
void set_delimiter(char delimiter_, std::string const& escape_str_);

// Get delimiter.
std::string const& get_delimiter() const;
```

#### Public member functions of ofstream and ocachedfstream (File stream for writing)

```cpp
// Open a text file for writing, if file exists, it will be overwritten.
void open(const std::string& file);
void open(const std::wstring& file);
void open(const char * file);
void open(const wchar_t * file);

// Query whether the file is opened successfully
bool is_open();

// Flush the contents to the file. To be called before close.
void flush();

// Close the file.
void close();
```

#### Public member functions of ostringstream (String stream for writing)

```cpp
// Get the text that has been writtten with the << operator
std::string get_text();
```

## FAQ
__Why do the reader stream encounter errors for csv with text not enclosed within quotes?__

Ans: To resolve it, Please remember to call enable_trim_quote_on_str with false.

## Common mistakes

Forget to call set_delmiter() for the next line after changing the delmiter on the fly with the sep class.

