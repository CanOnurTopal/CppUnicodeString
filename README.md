# Unicode String For C++

This is a library to process unicode text in C++. Primarily a STL like string class that can work on unicode text data.

Anyone who has processed text in C++ knows how annoying processing modern text can be. Get an input not in the ASCII standard (Which happens even in English) and the standard string starts behaving problematically. While the newer standards have introduced some helpful tools, it is still very lacking. 

For a production level code I recommend a localization library like ICU. However, considering localisation is time consuming/annoying, and can be overkill for a lot of purposes, I wanted to have a better alternative to jump-start a project.

That is why I created the utf8string class. It is designed to be very similar to the standard string class and light weight. The purpose is to abstract away character representation and focus on what you want to achieve. Copy the code into a project and start using it.

I'm making this public because since I needed it, other people probably need something like this too.




# Getting Started

If you know all about the Unicode Transformation Format (UTF) then feel free to skip the explanation. If not, here is a grossly oversimplified overview.

##  Unicode Transformation Format (UTF)

UTF is an widely (almost exclusively) used encoding method for unicode characters. The encoding maps to Unicode code points (for example 934 = Φ). UTF comes in 3 main flavours:

 - UTF-8
 - UTF-16
 - UTF-32
 
The numbers on the right show how many bits the encoding takes up. 

As you might have guessed, 8 and 16 bits are not enough to encode the entirety of the Unicode characters. That is why they are variable width. In UTF-8, if 8 bits are not enough, a second 8 bits is added and if that is not enough, a third 8 bits is added etc. So 'A' will take up 8 bits while '≧' will take up 24 bits. Same logic applies for UTF-16.

UTF-32 is fixed-width. No matter what the character, it will encode it using 32 bits. The pro here is that fixed width has performance advantages. Pointer arithmetic works as you would want it to. There is even some support for it in the C++ standard. The obvious con is that it wastes a lot of memory, particularly if you are working with a latin-script alphabet like English, German or Turkish. Using 4 times the memory to support the occasional umlaut is not efficient. 

It should be noted that for a lot of platforms (particularly the internet) uses UTF-8 as the de facto standard. Although you should double check when working on a file.

## Little insight on the Internals

Internally, the utf8string (as the name implies) uses utf8 to encode the text. This allows it to take advantage of standard C++ libraries (which tend to be more efficient) and saves memory.

However indexing and comparing the strings does involve more work since utf8 is not fixed width. So as to not iterate over the entire string to retrieve a character, utf8string separately stores the length of every character that cannot be represented by 1 byte. This is used to calculate the location of a character. While this is fine if you expect smaller characters, but if you are working with larger characters (like the Chinese alphabet), utf8string may end up using similar memory to UTF-32.


### String Literals

In C++, C style char string literals are commonly used in conjunction with the string class. So the standard library string has an extensive support for them. It should be noted that utf8string does not have explicit support for char string literals. So the **following code will not work**.

`utf8string test("This is a test string");`

While I might add support for this in the future, it does encourage a bit of recklessness that can lead to bugs. So instead there is support for char32_t string literals (which is utf32 literal). Simply add a capital U in front of your string literal and you are good to go. The **following code will work**

`utf8string test(U"This is a test string");`

### A small warning
Subscripting the utf8string will not return a reference to theunderlying utf8 representation. It will instead return a proxy class for a char32_t character. Since the underlying data is not the same data that is returned, and subscripting can also be used to assign new characters, subscripting will instead return a proxy class to support assignment.

This class can be cast to char32_t and is meant to be converted implicitly. However, keep in mind that explicit conversion may sometimes be required. In which case you can simply cast the class or use the ::data() function to get the char32_t value.

The underlying data is converted to char32_t the moment it is cast and not when the proxy class is created. Assigning a char32_t value to this proxy class will update the referenced character. 

On any assignment, compound assignment, increment and decrement operations, the proxy class will assign and return a char32_t instead a reference to itself. auto in `auto char = ++string[x];` will evaluate to char32_t. This means that the following ugly code will fail: `++string[x] = U'E';`. While this shouldn't present a lot of problems you should be aware of this.

Assigning value to a proxy class returned by a const utf8string will raise an error like "no matching overloaded function found" (depending on your compiler) since there is no const operator overload (and because you shouldn't be trying to update a const string). 

It is not recommended to store the proxy class as it does not check if the referenced value is valid.

If you simply want to read and store a character, utf8string::valueat(index) will return a char32_t value instead of the proxy.


# Manual

TODO

# Compatibility/Portability
I tried to make this class as portable as possible. However, there are a few assumptions. Most importantly:

**1 byte (aka char) must be at least 8 bits.** This is important if you are using non-standard extensions in your compiler.

While this is true for almost all modern computers, this is not guaranteed by the C++ standard. I didn't try to force this since I don't think anyone will have an issue with this. If you do have a problem, feel free to contact me.

Additionally, the class uses functionality intoduced in the C++11 specification. So your compiler should support that.

# TO-DO
 - Complete bug testing.
 - Test on windows.
 - Add ostream support
 - Add support for Unicode normalization
 - Complete support for all basic_string functions
 - Add getline overload
 - Add automatic encoding detection

 # Known Bugs
 - There may be issues with big endian computers/data. Didn't get a chance to test.



