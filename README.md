# extint

Ext(ernal) int(eraction): a Node.js module for using dynamically linked libraries straight from Node *without* performance penalty.
With this module you no longer have to create C++ wrappers for every library you have. Or better: you no longer have to recompile *all* your wrappers for every Node.js update.
This module is precompiled for Windows so you don't have to go through all the hassle. 
[Node-ffi](/rbranson/node-ffi) already provides this functionality, but has quite some call overhead.

## How
Extint procedure calls are just as fast as their hardcoded versions. Extint comes bundled with a mini-compiler.
Up on initialization of a library, (`extint.load()`), the procedure calls get compiled dynamically. For example:

    var kernel32 = extint("Kernel32.dll", {
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

* `void` : (`void`) - No value. (Only valid as a return type.)
* `bool`, `boolean` : (`bool`) - A type with only two value: `true` and `false`. (8 bit)
* `ptr` : (`void*`) - A memory pointer. Size depends on executing machine. (32-64 bit)
* `int8` : (`char`) - A 8 bit signed integer.

## Methods

### `extint.load(library, functions)`

#### Description
Loads a dynamically linkable library, gets the function pointers to the specified functions and wraps those pointers with some wrapper code.

#### Arguments
- library: Path to the library. (Usually .dll files on Windows).
- functions: A map with each key naming which function to exctract. Every key should be mapped to a array with two elements: the return type and another array containing the argument types. 
