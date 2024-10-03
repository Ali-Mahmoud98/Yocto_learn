# Classes and `.bbclass` files

### Introduction to Classes
Suppose you have multiple recipes for different software packages and all need similar build configurations such as fetching source code, configuring, compiling, and installing. So instead of writing the same code in each recipe you can use `.bbclass` file with the common tasks and fuctionalities then the recipes can inherit the `.bbclass` file to reuse its fuctionalities.

`.bbclass` files encapsulates reusable build metadata for the recipes. The classes are created in `.bbclass` files.

**Class** files are used to abstract common functionality and share it amongst multiple recipe (.bb) files. To use a class file, you simply make sure the recipe inherits the class. In most cases, when a recipe inherits a class it is enough to enable its features. There are cases, however, where in the recipe you might need to set variables or override some default behavior.

Any [Metadata](https://docs.yoctoproject.org/ref-manual/terms.html#term-Metadata) usually found in a recipe can also be placed in a class file. Class files are identified by the extension .bbclass and are usually placed in one of a set of subdirectories beneath the `meta*/`directory found in the [Source Directory](https://docs.yoctoproject.org/ref-manual/terms.html#term-Source-Directory):
* classes-recipe/ - classes intended to be inherited by recipes individually
* classes-global/ - classes intended to be inherited globally
* classes/ - classes whose usage context is not clearly defined

Class files can also be pointed to by [BUILDDIR](https://docs.yoctoproject.org/ref-manual/variables.html#term-BUILDDIR) (e.g. build/) in the same way as `.conf` files in the conf directory. Class files are searched for in [`BBPATH`](https://docs.yoctoproject.org/ref-manual/variables.html#term-BBPATH) using the same method by which `.conf` files are searched.

It is good to visit [Classes](https://docs.yoctoproject.org/ref-manual/classes.html) in the yocto documentation.

**NOTE**: You can find the `.bbclass` files inside the `classes` directory inside the **Meta Layer**.

### Example making your own `.bbclass` file
Inside your custom layer make a directory called classes then add your `.bbclass` file. for example you added a file called `newClass.bbclass` with a content like the following:
```bash
do_fetch_info(){
    bbplain "************************************"
    bbplain "*        Fetching ${PN}_${PV}      *"
    bbplain "************************************"
}

addtask do_fetch_info before do_fetch

do_unpack_info(){
    bbplain "************************************"
    bbplain "*        Fetching ${PN}_${PV}      *"
    bbplain "************************************"
}

addtask do_unpack_info before do_unpack
```

`bbplain` and `addtask` are related to the BitBake build system, which is the core of Yocto Project.

#### `bbplain`
`bbplain` is a command used to run a BitBake recipe in a plain format. It allows you to execute a recipe without the overhead of the full BitBake environment. This can be useful for debugging or testing specific parts of a recipe without going through the entire build process.

#### `addtask`
`addtask` is a BitBake function used to define a new task in a recipe. It allows you to specify a task that should be executed at a certain point in the build process. The syntax typically looks like this:
```bash
addtask <taskname> [before|after] <other_task>
```

For example, if you want to add a task called `mytask` that should run after the `do_compile` task, you would write:
```bash
addtask mytask after do_compile
```

#### Nested Class:
you can inherit a class file inside another class file. for example you have a `newClass2.bbclass`:
```bash
inherit newClass

do_between_fetch_and_unpack() {
    bbplain "****************************************"
    bbplain "*        Between fetch and unpack      *"
    bbplain "****************************************"
}

addtask do_between_fetch_and_unpack before do_unpack_info after do_fetch_info
```

#### Class in `.conf` file
you can use class in the `.conf` file using the following:
```bash
INHERIT += "newClass"
```
If I added this to the `local.conf` then the tasks inside `newClass.bbclass` will be added to all recipes.
