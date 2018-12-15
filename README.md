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

Adding more content
-------------------
A single web page is fine, but not really useful. We want to add another page.
So lets create a second _ecpp_ file. Create a file page2.ecpp with some html
content.

To include the new page to the application we add it to our _Makefile.am_. There
is a variable called _ecppSources_. Add it there. Note that you can put it just
after the _hello.ecpp_ separated by space or add a backslash at the end of the
line to continue on the next. See a autotools tutorial for more details if you
want.

When you now build the application with _make_, and run it, there is a new page,
which can be found under the url _http://localhost:8000/page2_. If you want, you
can add a link to that page into the _hello.ecpp_ and on _page2_ a link back to
_hello_. And note that you can find the hello page also under the url
_http://localhost:8000/hello_.

I think it is time to find out, why it is so. I already mentioned the mappings.
They tell tntnet, which page to call when a request arrives. Of course the url
is the key here.

Our pages are compiled with the _ecppc_ compiler. It gives all pages a name. By
default it is just the basename of the file, so that our _hello.ecpp_ is named
_hello_. It is the component id of the page. And similar _page2.ecpp_ is named
_page2_.

Now we look at the _main.cpp_. There we find calls to the method `mapUrl` of the
`tnt::Tntnet` object. It takes 2 parameters. A regular expression, which is
executed against a incoming url of a request and a component name, which to call
if the expression matches.

There are currently 3 mappings. We skip the first for now. The second has a
regular expression _^/$_. That matches exactly one slash. The url of a http
request start always with a slash. This is mapped to the component with the name
_hello_, which is our page generated from _hello.ecpp_.

The third mapping is a little more sophisticated. The expression is _^/(.*)_,
which actually matches everything. But the part after the first slash is
collected since it is in brackets. The component id, is _$1_, which is a back
reference to the first bracketed expression. So the url _/hello_ matches the
expression and _$1_ is set to _hello_ so that again our _hello.ecpp_ is called.
And _/page1_ calls _page1.ecpp_.

Now we come back to the first mapping. It maps static resources.

If you look at _Makefile.am_ you can find a rule to generate a component called
_resources_. The flag _-bb_ tells _ecppc_ to build a so called multi binary
component. It creates on component from many static source files. In those
source files no tags are interpreted. This is useful for all static files like
_css_ or _images_. It would be fatal if _ecppc_ would interpret a graphics image
and by chance find something, which looks like a _ecpp_ tag and tries to do
something with it.

Since now all static files have a single name _resources_ we somehow have to
tell tntnet, which file to fetch from those. If you look again at _main.cpp_ and
the first mapping you can see a call to the method `setPathInfo`. The method
`mapUrl` returns a mapping object, which can be modified.

The expression _/(.*)_ is the same as in the third mapping and hence matches
everything. This time the component to call is always _resources_ but the
mapping gets a additional information using this `setPathInfo`. In our case the
path is then _resources/$1_. So that the url _/hello.css_ sets the path info to
_resources/hello.css_. And indeed you can find such a file in our project and
also in our _Makefile.am_ under _staticSources_.

Note that _ecppc_ automatically sets a proper content type depending on the
extension of the file to make the browser happy.
