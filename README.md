# extint

Ext(ernal) int(eraction): a Node.js module for using dynamically linked libraries straight from Node *without* performance penalty.
With this module you no longer have to create C++ wrappers for every library you have. Or better: you no longer have to recompile *all* your wrappers for every Node.js update.
This module is precompiled for Windows so you don't have to go through all the hassle. 
[Node-ffi](/rbranson/node-ffi) already provides this functionality, but has quite some call overhead.



## How
Extint procedure calls are just as fast as their hardcoded versions. Extint comes bundled with a mini-compiler.
Up on initialization of a library, (`extint.load()`), the procedure calls get compiled dynamically. For example:

    var kernel32 = extint.library("Kernel32.dll", {
        'HeapAlloc': ['ptr', ['ptr', 'uint32', 'ptr']]
    });

A function equivalent to the following source code gets compiled on the fly.

    Handle<Value> proxy_HeapAlloc(const Arguments& arguments) {
        HandleScope scope;
        
        void* result = HeapAlloc(
            arguments[0]->UInt32Value(),
            arguments[1]->UInt32Value(),
            arguments[2]->UInt32Value()
        );
        
        return scope.Close( Number::New( result ) );
        
    }
    
Because of this, startup time may take longer, but in the long run it is faster.



## Datatypes

Format: `extint type` (`C++ equivalent type`) - Description.

* `bool, boolean` : (`bool`) - A type with only two values: `true` and `false`. (8 bit)
* `ptr` : (`void*, size_t`) - An unsigned integer big enough to hold a memory pointer on the machine it is running on. (32-64 bit)
* `int8` : (`char`) - A 8 bit signed integer.
* `int16` : (`short`) - A 16 bit signed integer.
* `int32` : (`int`) - A 32 bit signed integer.
* `int64` : (`long long, __int64`) - A 64 bit signed integer.
* `uint8` : (`unsigned char`) - A 8 bit unsigned integer.
* `uint16` : (`unsigned short`) - A 16 bit unsigned integer.
* `uint32` : (`unsigned int`) - A 32 bit unsigned integer.
* `uint64` : (`unsigned long long, unsigned __int64`) - A 64 bit unsigned integer.
* `float32` : (`float`) - A 32 bit IEEE 754 floating point number.
* `float64` : (`double`) - A 64 bit IEEE 754 floating point number.

The following types can only be used as return type:
* `void` : (`void`) - No value.

The following types can not be used as return type:
* `string:narrow` : (`const char*`) - On Windows the string is encoded using the system default Windows ANSI code page, which means that not all characters can be converted. Microsoft advises to not use the narrow version of their API anymore. On platforms other than Windows the string is encoded using UTF-8. 
* `string:wide` : (`const wchar_t*`) - On Windows the string is encoded using UTF-16. On other machines it is encoded using UTF-32.

### 64 bit integer return types

JavaScript internally stores numbers as 64 bit floating point numbers. Integers longer than 32 bits (`int64`, `uint64` and possibly `ptr`) can't be accurately represented as a floating point number. Therefore, **whenever extint predicts precision may get lost, it returns the integer as a string.** If you need to do some math with these numbers, use a bigint library.



## Methods

### `extint.library(path, functions)`

#### Description
Loads a dynamically linkable library, gets the function pointers to the specified functions and wraps those pointers with some wrapper code.

#### Arguments
- `path`: Path to the library. (Usually .dll files on Windows).
- `functions`: A map with each key naming which function to extract. Every key should be mapped to a array with two elements: the return type and another array containing the argument types. The maximum number of arguments per function is 32.

#### Example
    var user32 = extint.library("User32.dll", {
        'MessageBoxW': ['int32', ['ptr', 'string:wide', 'string:wide', 'uint32']]
    });
    
    user32.MessageBoxW(0, "Message", "Title", 0);



## Other information
This module is distributed with the [BSD license](LICENSE.md): you can do whatever you want with it, as long as you keep my name in it.

If you find a bug, please report it on the [Issues](https://github.com/DaveBakker/extint/issues) page.



# Todo

**Add support for the following string types:**
* `string:ascii` - An ASCII encoded string. Regardless of the operating system.
* `string:utf8` - An UTF-8 encoded string. Regardless of the operating system.
* `string:utf16` - An UTF-16 encoded string. Regardless of the operating system.


**Add support for directly calling functions by their function pointer:**

### `extint.function(functionPtr, signature)`

#### Description
Wraps a function pointer to make it callable from JavaScript.

#### Arguments
- `functionPtr`: A number which is to be interpreted as a pointer.
- `signature`: An array with two elements: the first element coontains a string describing the [return type](#datatypes). The second element contains an array with all the [argument types](#datatypes). The maximum number of arguments is 32.

#### Return value
Returns a JavaScript function which propagates the function call to the external function.

#### Example
    var wndProc = extint.function(wndProcPtr, ['ptr', ['ptr', 'uint32', 'uint32', 'ptr']]);
    
    wndProc(hwnd, WM_ERASEBKGND, dc, 0);

**Add a 'struct' type:**

### `extint.struct(layout)`

#### Example
    var Point = extint.struct({
        'x': 'int32',
        'y': 'int32'
    });
    
    var myPoint = new Point();
    myPoint.x = 10;
    myPoint.y = 55;
    
    console.log(myPoint.ptr);
    
