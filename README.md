Introduction to Tntnet
======================

Tntnet is a web application server for C++. The user can use C++ for developing
web applications.

Tntnet has a template language called _ecpp_, which allows the user to add C++
code into html pages. Those pages are preprocessed using the ecpp compiler
`ecppc`. The output are C++ classes, which are then compiled and linked into a
binary.

There are 2 modes of operation: standalone or module.

In standalone mode the user writes a `main` function, which instantiates a
tntnet instance, configures and runs it. The web server is so embedded into a
own application.

In module mode the templates are linked into a shared library, which is
dynamically loaded by the tntnet application.

Both modes can actually also be mixed since the tntnet instance can load those
modules also. It is just a matter of how it is configured.

First web application
---------------------
We assume that you have installed tntnet and for the build system a c++
compiler, autoconf, automake and libtool, since we those autotools are used to
build the projects. Other build tools can of course be used but for now lets
stay on autotools since the project templates use it.

Lets start to create a very simple web application. For a quick start we use a
script provided by tntnet called `tntnet-project`. This is a helper which
creates a tntnet project from a template.

So on the command line call:

    tntnet-project hello

This creates a project _hello_ in a directory _hello_ and initializes it. Change
to the directory and run:

    ./configure
    make

This will configure and build a program called _hello_. Run it with the command:

    ./hello

When you navigate with your browser to _http://localhost:8000/_ you see a simple
web page with a short greeting.

So lets look, what we have done. If you look at the directory there are quite
many files. Most of them are generated.

Look at the file _hello.ecpp_ first. It is our web page. It contains html code
with some additional tags, which are interpreted by the `ecppc` preprocessor.
Those tags are empty for now. They are just placeholders to give you an idea,
what you can do.

The next important file is _main.cpp_. This defines a `main` function as needed
in a C++ application. You can see, that it reads a configuration file
_hello.xml_ and instantiates a application object of type `tnt::Tntnet` with
that configuration. Next it adds some mappings to tell tntnet, what to do when a
request arrives.

The configuration file _hello.xml_ contains the settings for tntnet. The most
important and only mandatory part is the definition of the listeners. You have
to tell tntnet, on which port it should listen for incoming requests.

The build system needs a _configure.ac_ and _Makefile.am_. The _configure.ac_ is
the source for the `configure` script. The _Makefile.am_ tells the build system,
what to build. There you can find a reference to our _hello.ecpp_ and
_main.cpp_. Also there is a section for static resources. _hello.css_ can be
found there.
