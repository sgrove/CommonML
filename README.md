                                       CommonML

                          Easy OCaml Development and Sharing
                        Automatically builds dependent projects
                            Generates Autocomplete Support
                               Generates Documentation
                                    Just Hit Build



- `CommonML` allows you to build OCaml programs on `CommonJS/package.json`.
If you know `CommonJS/package.json`, there's not much to learn (besides OCaml).
- Dependencies are installed to the standard local directory `node_modules` sandbox.
- Your project (and dependencies) are automatically built in the correct order, into
a local artifact directory.
- All of your dependencies' `"exports"` are automatically namespaced via the package
name. If your `package.json` has two dependencies (`Package1` and `Package2`), and
each exports a `Util` module, `CommonML` automatically generates module
aliases that requires you to refer to them as `Package1.Util` and `Package2.Util`.
Internal modules within `Package1` and `Package2` may reference their internal
util module via `Util` (without namespace).
- Developing/depending on local packages is the same as developing against
remote dependencies. (Just use the standard `npm link` command).
- `CommonML` autogenerates autocompletion for internal modules and dependencies
in addition to docs.

>> This is merely an experiment that explores an OCaml development workflow based
on the familiar `CommonJS`.  `OPAM` is the official, high performance package
manager for OCaml and you should use that instead of this project for real development.
This is only intended for people who really want to try out OCaml development with
their familiar `CommonJS` workflow/namespacing.


**Automatically namespaces dependencies and generates docs**

<img src="https://raw.githubusercontent.com/jordwalke/CommonML/master/img/CommonMLDoc.png" />

**Automatically generates autocomplete for dependencies**

<img src="https://raw.githubusercontent.com/jordwalke/CommonML/master/img/CommonMLAutoComplete.png" />



Note: `npm` is a hosting service, in *addition* to a command line interface to `CommonJS/package.json`.
You can use only the command line tools and host your dependencies at arbitrary
git URLS (a reason why `CommonJS` is great). Nothing stops you from hosting
OCaml code on `npm`'s hosted service too. The `npm` command line is just a tool for
installing files on your disk.

Requirements
------------
- *Very* new version of nodeJS http://nodejs.org
- OCaml: Just install `OPAM` which installs OCaml too: http://opam.ocaml.org/doc/Install.html


If you already know `CommonJS`, you already know `CommonML`. Most of this `README`
is just a basic tutorial on `package.json`/`npm`.


00. Try Building an Example Project!
---------------
In `ExampleProjects/MyProject/` there is an example project (which just means
a `src/` directory with OCaml files and a `package.json` listing its dependencies).
Check out the `package.json` file to see that it depends on a couple of other
`CommonML` projects that are hosted on github (not npm's service).

    # Install the project's dependencies.
    cd ./ExampleProjects/MyProject/
    npm install   #Installs dependencies.

    # Build and Run
    node node_modules/CommonML/build.js
    ./_build_ocamlc/MyProject/myProject.out

    # Build some docs (open the link it generates)
    node node_modules/CommonML/build.js --doc=html

    # If you have OCaml's Vim/Emacs Merlin plugin installed, editing
    # any of the example files will now have working autocomplete.



1. Make A Package From Scratch:
-----------------

To make your own package just like the example:

    #1. Make a directory with a `package.json` file and list any modules
    # that should be visible to *other* packages in the "exports" field.

    mkdir MyProject && cd MyProject
    echo '{
      "dependencies": {"CommonML": "git://github.com/jordwalke/CommonML.git"},
      "CommonML": {
        "exports": ["MyPublicModule"],
        "compileFlags": [],
        "linkFlags": []
      }
    }' >> ./package.json

    #2. Downloads dependencies including `CommonML` into `./node_modules`
    npm install

    #3. Place all your package's `.ml` files inside of a directly named `src/`.
    mkdir ./src         # All .ml/.mli source files go here.
    echo 'let _ = print_string "Hello World"' >> src/MyPublicModule.ml

    #4. Build native executable
    node node_modules/CommonML/build.js

    #5. Run executable
    _build/MyProject/myProject.out


2. Building
-----------------------------

- All `CommonML` dependencies automatically built in the correct order.
- Every `CommonML` package now contains an autogenerated `.merlin` file for
  code-completion.
- Source directories free of build artifacts, mirrored into `MyProject/_build`.
- Executable output for topmost project placed into `MyProject/_build/MyProject/myProject.out`.

4. Depend On Other Packages
---------------------------
Depending on other `CommonML` packages is done exactly the same way as adding
any other `npm` dependencies.

1. Add any other valid `CommonML` packages to `dependencies`
2. Now your `package.json` looks like Figure 1 below.
3. Rerun `npm install`. This installs all dependencies into `node_modules`.
4. Now your package's source files can simply refer to `SomeOtherPackage.TheirExportedModules.blah`.




Figure 1:


      "dependencies": {
         "CommonML": "git://github.com/jordwalke/CommonML.git",
         "SomeOtherPackage: "git://github.com/jordwalke/SomeOtherPackage.git"
       },


Sharing:
-------

  1. Just push your `package.json` and `src/` directory to any git URL.
  2. Other people may point to *that* git URL in *their* package.json's dependencies.
  3. Other people just run `npm install` and your package will be installed into *their* `node_modules` sandbox.

Resolving Duplicate Packages:
----------------------------
Duplicate modules are namespaced automatically, but duplicate packages can only be resolved
by the package manager - `npm`.
If two packages depend on two different versions of the same module - there is no solution yet.
If two packages depend on *compatible* versions of the same module, then `npm dedupde` should take care of it.

Build Parameters:
-----------------

Dedicated build directories (with their own resource caches) are created for
each build flag combination. For example, if you build a root project
`YourProject` "for debugging", with `ocamlopt`, then the resulting binary is
placed at `RootOfYourProject/_build_ocamlopt_debug/YourProject/yourProject.out`
and intermediate build artifacts for dependencies are placed at
`RootOfYourProject/_build_ocamlopt_debug/ADependency/...`.


*Build bytecode (default):(From `YourProject` root)*

    node node_modules/CommonML/build.js --compiler=ocamlc # To compile bytecode

    # Then run
    ./_build_ocamlc/YourProject/yourProject.out


*Build native binaries:(From `YourProject` root)*

    node node_modules/CommonML/build.js --compiler=ocamlopt # To compile to native binaries

    # Then run
    ./_build_ocamlopt/YourProject/yourProject.out


*Build beautiful docs:(From `YourProject` root)*

    # Add the [--doc] command to any build command
    # valid options are `html`, `latex`, `texi`, `man`, `dot`.
    # Recursively builds documentation for all dependencies as well.
    node node_modules/CommonML/build.js --compiler=ocamlc --doc=html
    open _build/YourProject/yourProject.doc/index.html


*To enable stack traces:(From `YourProject` root)*

    # Your binary and dependencies must be compiled with --forDebug
    node node_modules/CommonML/build.js --forDebug=true
    # You must have a runtime parameter set
    export OCAMLRUNPARAM='b'
    # Execute the binary that was built into the dedicated directory
    ./_build_blah_blah/YourProject/yourProject.out


*Build JavaScript With Sourcemaps:(From `YourProject` root)*

> If you have feedback about the actual debugging experience with source maps
> (breakpoints, stepping through etc), please file issues on the `js_of_ocaml`
> github page. Include screenshots, screencasts or detailed descriptions of how
> the debugging experience can be made intuitive and more similar to debugging
> JS.
>
>     https://github.com/ocsigen/js_of_ocaml/issues


    # Building JavaScript is a special case of building for debug, and building
    # for bytecode (a final step converts the bytecode to JavaScript).
    # Ensure you have `js_of_ocaml` installed and available on your PATH.
    # Install via `opam install js_of_ocaml`.
    node node_modules/CommonML/build.js --forDebug=true --jsCompile=true
    # open the test page

    # Then run
    open ./_build_ocamlc/YourProject/yourProject.html


Customizing JavaScript Build Location And Running With Server
----------------------------


`--jsCompile` will generate two artifacts: The symlink (`jsBuild`) to the
actual js bundle and source maps and a fake directory structure to get
source maps to work when served from a web server.

When building for JS, the root package's `package.json`'s
`CommonML.jsPlaceBuildArtifactsIn` field will determine where the js build
artifacts will be placed. It is considered relative to package root.  If not
specified, it will default to the package root itself.

When running locally, source maps naturally work correctly because they
specify absolute file paths to original source files. But when serving that
web server root from a web server, the same origin policy restricts you
from seeing the source maps. So (unfortunately) `CommonML` has to pollute
that destination `jsPlaceBuildArtifactsIn` with a faked directory path that
resembles the absolute path to where your original source files were
located so that source maps will work even when running on a web server.
You will see a generated path that symlinks to your original build
directory, such as:


    YourProject/myJsPlaceBuildArtifactsIn/Users/yourName/path/to/YourProject/_build_ocamlc_debug ->  YourProject/_build_ocamlc_debug/


This is just so that when you run a web server at
`YourProject/myJsPlaceBuildArtifactsIn` (or whatever you configured
`jsPlaceBuildArtifactsIn`) source maps will work correctly.



Integrating With JavaScript:
----------------------------

When you build for JavaScript `--forDebug=true --jsCompile=true`, you are
simply compiling into JS. This doesn't give you the ability to do anything
except print output. To actually interface with the containing JavaScript
environment, you'll need to not only *compile* using `js_of_ocaml`, but also
use the `js_of_ocaml` runtime libraries which should be compiled into
JavaScript along with your application code. To ensure that it is included, and
linked, add `js_of_ocaml` to your `package.json` file's
`CommonML.findlibPackages` field.  (There's also a syntax extension available
to make interacting with JavaScript more sugary).


    "CommonML": {
      "findlibPackages": [{"dependency": "js_of_ocaml"}],


You can also include a field in your `package.json`'s `CommonML` section called
`jsResources`. This may be set to a directory (relative to your project's root)
whos contents should be copied into the js build directory. A generated
`app.js` file will also be generated by `js_of_ocaml` and placed alongside
them. See `ExampleProjects/MyProject/`.


    "CommonML": {
      "jsResources": "jsResourcesDir",


`CommonML` ensures that source maps work whether or not you serve your files
from a web server, or from local disk. It does this by creating the appropriate
sym links that your browser's source maps will understand.

TODO: `js_of_ocaml` should itself be turned into a `CommonML` dependency to
remove any need to use the `findlibPackages` field (it's confusing that we even
have the notion of "findlib packages") and `package.json` is fully sufficient
to do everything we need. This is only temporary.

Compatibility:
-------------

  1. CommonML is opinionated and only builds packages of a certain form.
  2. See the documentation for more details.
  3. Each package lists all of its dependencies in its package.json.
  4. CommonML ensures that each of your dependency\' packages are accessible
     and build before your package.
  5. Each of your dependencies must also be CommonML compatible.

Why NPM?:
--------

  1. npm (command line tool, that CommonML uses) is *not* npmjs.com (service).
  2. npm (command line tool) is merely a way to organize and install dependencies - it has nothing to do with JS.
  3. npm do not depend on any central repository and is extremely popular.
  4. npm (and therefore CommonML) can work entirely based on github repos.
  5. npm allows local development as a special case of sharing (see `npm link`)


Package Structure
-----------------

Any set of files/directories that have the following form are a valid CommonML
project. To make a CommonML project, just make these files with the following:

- Root project directory that matches the name of your project. In this case
  `MyProject`.
- A `src` directory that may contain any files, but may not contain two `.ml`
  files with the same file name.
- A `package.json` file directly inside of the project root directory (more on
  that later).


Typical project structure.

    └── MyProject/
        ├── package.json
        └── src/
            ├── moduleOne.mli
            ├── moduleOne.ml
            └── someDirectory/
                └── someOtherModule.ml


Recall that in OCaml, a file named `foo.ml` automatically becomes a module
`Foo`. The only question is, which modules can see other modules? CommonML
comes up with a reasonable convention that makes sharing easy.

Suppose you hae a package root directory named `YourProject/`
- The module located at `YourProject/src/any_path/moduleX.ml` can access
  `YourProject/src/any_other_path/moduleY.ml` simply by typing `ModuleY`. The
  file paths don't effect visibility *within* a package.
- Therefore no two OCaml files inside of `YourProject/src` may have the same
  name -> no two OCaml modules inside of `YourProject/src` may have the same
  name.
- `YourProject` package allows other dependent packages to reference internal
  module `YourInternalModule` if and only if `YourInternalModule` name is
  listed in `YourProject`'s `package.json` (`CommonML.exports` field).

The `package.json`:
-------------------

Each package's `package.json` must provide information that instructs CommonML
how to compile, run, and be depended on by other packages.

    {
      "name": "MyProject",
      "version": "1.0",
      "description": "Simple My Project example in OCaml",
      "dependencies": {
        "CommonML": "ssh+git://github.com/jordwalke/CommonML.git",
        "YourProject": "1.0"
      },
      "CommonML": {
        "exports": ["ExportedModule", "AnotherExportedModuleName"],
        "compileFlags": ["-g", "-w","-30","-w","-40"],
        "findlibPackages": [
          {"dependency": "comparelib"},
          {"syntax": "comparelib.syntax"}
        ],
        "docFlags": ["-css-style", "docStyle.css"]
      }
    }


- There is also a place to put compiler flags and `findlibPackages` in the
  `package.json`.


Use CommonML To Tame Local Development:
--------------------------------------

You can use the familiar `npm link` command to have projects depend on other
projects locally without publishing them publicly.

    packages/
    ├── MyProject/
    │   ├── package.json      // { "dependencies": {"YourProject": "1.0", "CommonUtility": "1.0"},
    │   └── src/              //    "exports: ["MyProject", "Util"]
    │       ├── myProject.mli // }
    │       ├── myProject.ml
    │       └── util/
    │           └── util.ml
    ├── YourProject/
    │   ├── package.json     // {"dependencies": "ObscureUtility": "1.0", "CommonUtility": "1.0"}
    │   └── src/
    │       ├── yourProject.mli
    │       ├── yourProject.ml    // Only visible to MyProject
    │       └── util/
    │           └── util.ml       // May only be observed by modules in MyProject - distinct from MyProject.util
    ├── CommonUtility/
    │   ├── package.json     // {"dependencies": {}
    │   └── src/
    │       ├── yourProject.mli
    │       ├── yourProject.ml      // Only visible to MyProject
    │       └── yourUtils.ml        // May only be observed by modules in YourProject
    └── ObscureUtility/
        ├── package.json     // {"dependencies": {}}
        └── src/
            ├── obscureUtility.mli
            └── obscureUtility.ml  // May only be observed by YourProject



Goals:
======

- Local development should be the same exact workflow as sharing and depending
  on published modules. Local development is merely sharing with yourself.

- No `README` should ever contain the phrase: "Make sure you have package X
  installed globally on your system".

- Installing a package should *always* be as simple as listing it as a
  dependency in `package.json` and running `npm install`. It should install
  everything you need to be able to build, run and generate documentation.
  This isn't always possible for huge projects, but `CommonML` is for the
  subset of projects for which it *is* possible.

*Yes, `CommonML` currently relies on having `OCaml/nodeJS` installed on your system.
One thing at a time.


FAQ:
===

Which module is the root of the executable? `CommonML`'s build script
automatically determines this by way of using `ocamldep` behind the scenes. In
short, it's whichever one does not have any other dependencies.



Preprocessors:
==================

You may supply a custom preprocessor program that should be executed on each
file in order to parse the source into a canonical AST. The preprocessor must be
available in your `PATH` and be able to accept an arbitrary file, and either
invoke a custom parser, or invoke the standard parser. Your preprocessor likely
would use file extensions as "hints" as to which to do.

Therefore, `CommonML` also allows you to indicate that particular "pairs" of
extensions should be treated as interfaces/implementations respectively -
otherwise the file extension "hints" that are used by your preprocessor will
confuse the rest of the compilation toolchain.

You will often want to supply both a preprocessor *and* extensions. Below is an
example of doing both, by populating the `preprocessor` and `extensions` fields
in a `package.json`'s `CommonML` field:

      "CommonML": {
        "exports": ["MyProjectMod"],
        "preprocessor": "myCustomPreprocessor",
        "extensions": [{
          "interface": ".heyoi",
          "implementation": ".heyo"
        }],

The `preprocessor` and `extensions` fields are separate concepts that are often
used simultaneously.

TODO:
=====

  - Replicate this development flow with OPAM/ocamlbuild to see how
    close we can get.
  - Different configurable compiler flags for ocamlc vs ocamlopt.
  - Warn when observing .ml/i files outside of `src` directory.
  - Build `CommonML` compatible packages from OPAM.
  - Generate ocamldebug startup script for last dependency seen in ocamldep,
    set to break on the first line.
  - An `a.out` is generated by `ocamlfind` when it shouldn't be: `ocamlfind
    ocamlc -linkpkg -package js_of_ocaml -only-show` is supposed to find
    transitive dependencies, but only show without compiling, but it compiles
    and generates a left-over `a.out`. This is easily reproducable.
  - The `--forDebug` flag should be examined to determine if the optimized js
    compilation should be used (and source maps omitted).
  - Find a better solution for sourcemaps than creating fake directory
    structures and/or have `js_of_ocaml` just inline the source contents into
    the sourcemaps so we don't have to deal with file paths at all.



OTHER:
=====

*To show OCaml parsing errors :*

    export OCAMLRUNPARAM='p'


Stylesheet for documentation borrowed from vim-awesome (MIT license).
