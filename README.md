# PGLoader

pgloader is a data loading tool for PostgreSQL, using the `COPY` command.

Its main avantage over just using `COPY` or `\copy` and over using a
*Foreign Data Wrapper* is the transaction behaviour, where *pgloader* will
keep a separate file of rejected data and continue trying to `copy` good
data in your database.

The default PostgreSQL behaviour is transactional, which means that any
erroneous line in the input data (file or remote database) will stop the
bulk load for the whole table.

pgloader also implements data reformating, the main example of that being a
transformation from MySQL dates `0000-00-00` and `0000-00-00 00:00:00` to
PostgreSQL `NULL` value (because our calendar never had a *year zero*).

## Versioning

The pgloader version 1.x from a long time ago had been developped in `TCL`.
When faced with maintaining that code, the new emerging development team
(hi!) picked `python` instead because that made sense at the time. So
pgloader version 2.x were in python.

The current version of pgloader is the 3.x series, which is written in
[Common Lisp](http://cliki.net/) for better development flexibility, run
time performances, real threading.

The versioning is now following the Emacs model, where any X.0 release
number means you're using a development version (alpha, beta, or release
candidate). The next stable versions are going to be `3.1` then `3.2` etc.

When using a development snapshot rather than a released version the version
number includes the git hash (in its abbreviated form):

  - `pgloader version "3.0.99"`
  
     Release candidate 9 for pgloader version 3.1, with a *git tag* named
     `v3.0.99` so that it's easy to checkout the same sources as the
     released code.
     
  - `pgloader version "3.0.fecae2c"`
  
     Development snapshot again *git hash* `fecae2c`. It's possible to have
     the same sources on another setup with using the git command `git
     checkout fecae2c`.

  - `pgloader version "3.1.0"`
  
     Stable release.

## INSTALL

pgloader is now a Common Lisp program, tested using the
[SBCL](http://sbcl.org/) (>= 1.1.14) and
[Clozure CL](http://ccl.clozure.com/) implementations with
[Quicklisp](http://www.quicklisp.org/beta/).

    $ apt-get install sbcl libsqlite3-dev make curl
    $ cd /path/to/pgloader
	$ make pgloader
	$ ./build/bin/pgloader --help

You can also fetch pre-made binary packages at
[pgloader.io](http://pgloader.io/download.html).

## Testing a new feature

Being a Common Lisp program, pgloader is able to *upgrade itself* at run
time, and provides the command-line option `--self-upgrade` that just does
that.

If you want to test the current repository version (or any checkout really),
it's possible to clone the sources then load them with an older pgloader
release:

    $ /usr/bin/pgloader --version
    pgloader version "3.0.99"
    compiled with SBCL 1.1.17
    
    $ git clone https://github.com/dimitri/pgloader.git /tmp/pgloader
    $ /usr/bin/pgloader --self-upgrade /tmp/pgloader --version
    Self-upgrading from sources at "/tmp/pgloader/"
    pgloader version "3.0.fecae2c"
    compiled with SBCL 1.1.17

Here, the code from the *git clone* will be used at run-time. Self-upgrade
is done first, then the main program entry point is called again with the
new coded loaded in.

## The pgloader.lisp script

Now you can use the `#!` script or build a self-contained binary executable
file, as shown below.

    ./pgloader.lisp --help

Each time you run the `pgloader` command line, it will check that all its
dependencies are installed and compiled and if that's not the case fetch
them from the internet and prepare them (thanks to *Quicklisp*). So please
be patient while that happens and make sure we can actually connect and
download the dependencies.

## Build Self-Contained binary file

The `Makefile` target `pgloader` knows how to produce a Self Contained
Binary file for pgloader, named `pgloader.exe`:

    $ make pgloader

By default, the `Makefile` uses [SBCL](http://sbcl.org/) to compile your
binary image, though it's possible to also build using
[CCL](http://ccl.clozure.com/).

    $ make CC=ccl pgloader

Note that the `Makefile` uses the `--compress-core` option when using SBCL,
that should be enabled in your local copy of `SBCL`. If that's not the case,
it's probably because you did compile and install `SBCL` yourself, so that
you have a decently recent version to use. Then you need to compile it with
the `--with-sb-core-compression` option.

You can also remove the `--compress-core` option that way:

    $ make COMPRESS_CORE=no pgloader

The `--compress-core` is unique to SBCL, so not used when `CC` is different
from the `sbcl` value.

The `make pgloader` command when successful outputs a `./build/bin/pgloader`
file for you to use.

## Usage

Give as many command files that you need to pgloader:

    $ ./build/pgloader.exe --help
    $ ./build/pgloader.exe <file.load>
	
See the documentation file `pgloader.1.md` for details. You can compile that
file into a manual page or an HTML page thanks to the `pandoc` application:

    $ apt-get install pandoc
	$ pandoc pgloader.1.md -o pgloader.1
	$ pandoc pgloader.1.md -o pgloader.html
