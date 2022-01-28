# seL4-DevKit-Doc
mdBook documentation for seL4 DevKit

## Building the documentation with mdbook
The sel4devkit documentation is able to be viewed [here](seL4-doc/src/SUMMARY.md) using the built in GitHub markdown viewer, but is also designed to be built using the mdbook command line tool.

### Getting mdbook
mdbook can be downloaded from the releases section of its GitHub repository, which can be found [here](https://github.com/rust-lang/mdBook). Installation instructions can be found in the mdbook documentation [here](https://rust-lang.github.io/mdBook/).

### Building the sel4devkit documentation (HTML)
Once you have installed mdbook, the following instructions can be followed to create a HTML version of the documentation:

1. Open a terminal.
2. In a suitable location, clone this repository using git.
3. From the `seL4-doc` folder within the main repo folder, run `mdbook build`.
4. mdbook will now compile HTML from the markdown within the src folder.
5. Once mdbook has finished compiling, a folder called `book` will be created within the `seL4-doc` folder, containing the compiled HTML version of the documentation.
6. Open the `index.html` file contained within the `book` folder using a browser to view the documentation.
