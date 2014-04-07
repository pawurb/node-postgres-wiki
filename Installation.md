# Module Installation

To install `pg`, run the following:

`npm install pg`


# Dependencies for Native Bindings

## Ubuntu
Installation of this module via `apt-get` requires native bindings; and compilation is required even if you do not plan to use the native interface.  Thus the following command is mandatory.

To install the dependencies in Ubuntu:
`sudo apt-get install libpq-dev build-essential`

The above also appears to be sufficient.  Specifically, a working installation of PostgreSQL is not required on the same machine running Node.
