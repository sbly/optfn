optfn [![Build Status](https://travis-ci.org/ysimonson/optfn.png)](https://travis-ci.org/ysimonson/optfn)
=====

optfn uses introspection to make a Python function available as a command
line utility. It's syntactic sugar around optparse from the standard library.

optfn was built off of [simonw's optfunc](https://github.com/simonw/optfunc),
however they are incompatible with one another - optfn adds support for
varargs and python3, and removes class-based sub-commands.

Here's what the API looks like:

    import optfn

    def upper(filename, verbose=False):
        "Usage: %prog <file> [--verbose] - output file content in uppercase"

        s = open(filename).read()
        
        if verbose:
            print "Processing %s bytes..." % len(s)
        
        print s.upper()

    if __name__ == '__main__':
        optfn.run(upper)

And here's the resulting command-line interface:

    $ python demo.py --help
    Usage: demo.py <file> [--verbose] - output file content in uppercase
    
    Options:
      -h, --help     show this help message and exit
      -v, --verbose  

    $ python demo.py README.md 
    OPTFN
    ...

    $ python demo.py README.md -v
    Processing 2049 bytes...
    OPTFN
    ...

How arguments work
------------------

Non-keyword arguments are treated as required arguments - optfn.run will 
throw an error if they number of arguments provided on the command line 
doesn't match the number expected by the function (unless @notstrict is used, 
see below).

Keyword arguments with defaults are treated as options. At the moment, only 
string and boolean arguments are supported. Other types are planned.

Consider the following:

    def geocode(s, api_key='', geocoder='google', list_geocoders=False):

's' is a required argument. api_key, geocoder and list_geocoders are all 
options, with defaults provided. Since list_geocoders has a boolean as its 
default it will be treated slightly differently (in optparse terms, it will 
store True if the flag is provided on the command line and False otherwise).

The command line options are derived from the parameter names like so:

    Options:
      -h, --help            show this help message and exit
      -l, --list-geocoders
      -a API_KEY, --api-key=API_KEY
      -g GEOCODER, --geocoder=GEOCODER

Note that the boolean --list-geocoders is a flag, not an option that sets a
value.

The short option is derived from the first letter of the parameter. If that 
character is already in use, the second character will be used and so on.

The long option is the full name of the parameter with underscores converted 
to hyphens.

If you want complete control over the name of the options, simply name your 
parameter as follows:

    def foo(q_custom_name=False):

This will result in a short option of -q and a long option of --custom-name.

Special arguments
-----------------

Kwargs with the names 'stdin', 'stdout' or 'stderr' will be automatically 
passed the relevant Python objects, for example:
    
    #!/usr/bin/env python
    # upper.py
    import optfn

    def upper_stdin(stdin=None, stdout=None):
        stdout.write(stdin.read().upper())

    if __name__ == '__main__':
        optfn.run(upper_stdin)

Does the following:

    $ echo "Hello, world" | ./upper.py
    HELLO, WORLD

Subcommands
-----------

Some command line applications feature subcommands, with the first argument 
to the application indicating which subcommand should be executed.

optfn supports this - you can pass an array of  functions to `optfn.run()` and
the names of the functions will be used to select a subcommand based on the
first argument:

    import optfn
    
    def one(arg):
        print "One: %s" % arg
    
    def two(arg):
        print "Two: %s" % arg
    
    def three(arg):
        print "Three: %s" % arg
    
    if __name__ == '__main__':
        optfn.run([one, two, three])

Usage looks like this:

    $ ./subcommands_demo.py    
    Unknown command: try 'one', 'two' or 'three'
    $ ./subcommands_demo.py one
    one: Required 1 arguments, got 0
    $ ./subcommands_demo.py two arg
    Two: arg

This approach is limited in that help can be provided for an individual option 
but not for the application as a whole. If anyone knows how to get optparse to
handle the subcommand pattern please let me know.

Decorators
----------

optfn also supports two decorators for extra functionality:

    @optfn.notstrict
    @optfn.arghelp('list_geocoders', 'list available geocoders and exit')
    def geocode(s, api_key='', geocoder='google', list_geocoders=False):
        # ...

@notstrict means "don't throw an error if one of the required positional 
arguments is missing" - in the above example we use this because we still want
the list_geocoders argument to work even if a string has not been provided.

@arghelp('arg-name', 'help text') allows you to provide help on individual 
arguments, which will then be displayed when --help is called.
