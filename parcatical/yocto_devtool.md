# Devtool
As mentioned earlier, devtool helps you easily develop projects whose build output must be part of an image built using the OpenEmbedded build system. The remainder of this section presents the workflow you would use that includes devtool.

The devtool command-line tool provides a number of features that help you build, test, and package software. This command is available alongside the bitbake command. Additionally, the devtool command is a key part of the extensible SDK.

It is good to read [devtool in yocto documentation](https://docs.yoctoproject.org/ref-manual/devtool-reference.html) and [Using `devtool` in Your Workflow](https://docs.yoctoproject.org/1.8.2/dev-manual/dev-manual.html#using-devtool-in-your-workflow).

## Contents:
* [1. What is devtool?](#1-what-is-devtool)
* [2. Examples of using `devtool`](#2-examples-of-using-devtool)
* [3. Create Recipe using `devtool` and Deploy the build output on target](#3-create-recipe-using-devtool-and-deploy-the-build-output-on-target)
* [4. Patch with `devtool modify`](#4-patch-with-devtool-modify)
* [5. Upgrading recipe using `devtool upgrade`](#5-upgrading-recipe-using-devtool-upgrade)

### 1. What is devtool?
Yocto provide another tool like `bitbake` called `devtool` for the developers. the `devtool` is used for three main reasons which are add, modify, and upgrade.
* we can use `devtool` to create a recipe.
* we can use `devtool` to modify the sources and make patches.
* we can upgrade our recipes with `devtool`.

The three basic subcommands to work with a recipes are:
* add
* modify
* upgrade

you can read the `devtool` help
```bash
devtool -h
```

the `devtool` subcommands are used to make a patch:
* modify
* update-recipe 

To see how to use the `devtool` subcommands use:
```bash
devtool subcommand --help
```

### 2. Examples of using `devtool`
the basic command if you want to add a recipe:
```bash
devtool add hello-dev github-source-url # this will create a workspace if it is not created and add folder called
# hello-dev under folder called recipes and inside that folder the hello-dev_git.bb recipe is added
```

inside the workspace you will find four folders:
* appends: Holds `.bbappend` files to modify existing recipes. here are the recipes from `devtool upgrade` or `devtool modify`
* conf: Contains configuration files like `local.conf` and `bblayers.conf`.
* recipes: Contains your custom `.bb` recipe files. here you can find all the new recipes if you do `devtool add`
* sources: Holds local source code for custom packages. here the sources are unpacked.

you can build the recipe that you added using:
```bash
devtool build hello-dev
```
after building go to the `workspace/sources/hello-dev/` then see the content of it.

The following photo show the `devtool` workflow:

<p align="center">
    <img src="Screenshot from 2024-10-03 17-01-21.png" alt="Devtool Workflow"/>
</p>
<p align="center">
    This photo adapted from <a href="https://www.youtube.com/watch?v=Apfwyf_yEzI&list=PLwqS94HTEwpQmgL1UsSwNk_2tQdzq3eVJ&index=57">this YouTube video</a> at 0:39.
</p>

when you add a recipe using `devtool` the `devtool` fetch the source then create a recipe according to the source. for example, if there is a make file in a the source the `devtool` will create a recipe based on `OE_RUNMAKE` and if there is a cmake then the recipe will be created accordingly and so on.

### 3. Create Recipe using `devtool` and Deploy the build output on target
```bash
source oe-init-build-env
devtool add mydevtool-recipe github-url --srcbranch=branch_name # you will find a workspace directory in build directory
# open the bblayers.conf then you will find a new layer added
devtool build mydevtool-recipe
```
you may need to override the `CC`, `CFLAGS`, and etc using `EXTRA_OEMAKE`. Example
```bash
EXTRA_OEMAKE = "CC='${CC}' CFLAGS='${CFLAGS}'"
```

after building successfully then you can deploy the output into our target:
```bash
# connect the target to a LAN then see the ip using "ifconfig" or "ip a"
devtool deploy-target mydevtool-recipe root@${target_ip} # use this command inside the workspace directory
```

After the command worked well then undeploy the target:
```bash
devtool undeploy-target mydevtool-recipe root@${target_ip} # use this command inside the workspace directory
```

after that we need to finalize our recipe and need to put it in our meta layer.
```bash
devtool build mydevtool-recipe # the build here finishes succesfully
# but if you built only with devtool it does not do the packaging and all that stuff
# so we need to test the build using bitbake after build using devtool 
bitbake mydevtool-recipe # if there is error see the error
# there is a common error contains: does not have GNU_HASH (did not pass LDFLAGS)
# to solve the error add the following to EXTRA_OEMAKE
# EXTRA_OEMAKE = "CC='${CC}' CFLAGS='${CFLAGS} -Wl,--hash-style=gnu'"
# if the all error solved then you will finish the recipe
devtool finish --help # see how to use finish
devtool finish mydevtool-recipe ../meta-custom/recipes-app # you may need to add "-f" after finish
# check if the new recipe (mydevtool-recipe) added to meta-custom/recipes-app
rm -rf workspace
vim conf/bblayers.conf # delete the workspace layer
# you can use bitbake to check build after adding the new recipe
bitbake mydevtool-recipe
```

### 4. Patch with `devtool modify`
<p align="center">
    <img src="Screenshot from 2024-10-03 20-36-46.png" alt="Devtool Workflow"/>
    <img src="Screenshot from 2024-10-03 20-36-30.png" alt="Devtool Workflow"/>
</p>

<p align="center">
    Patching using devtool
</p>

Steps using commands:
```bash
devtool modify recipe-name
# go to the workspace/sources/recipe_name using devshell or in the same shell using using cd command
# using cd: $cd workspace/sources/recipe_name
# using devshell: $bitbake -c devshell recipe_name

# make your modifications

# add and commit your changes using git
git add your_changed_file

git commit -m "commit message" --signoff

# go to your build directory then do the following command
devtool update-recipe recipe-name -a /path/to/your/custom_layer

devtool reset recipe-name

vim conf/bblayers.conf # then delete the workspace layer

rm -rf workspace
```

### 5. upgrading recipe using `devtool upgrade`
you can upgrade your recipe to use a specific commit higher than the used commit:
```bash
devtool upgrade recipe-name -S commit_ID -V 2.0.1 # this line will update the SRCREV with the new commit ID and PV to be 2.0.1
# the upgraded source will be in "workspace/sources/recipe-name/" and upgraded recipes will be in "/workspace/recipes/recipe-name/recipe-name.bb"
vim workspace/recipes/recipe-name/recipe-name.bb # see the changes
devtool rename recipe-name -V 2.0.1
ls # see the file name, you should recipe-name_2.0.1

# check the sources in "workspace/sources/recipe-name/" and check the chages of commit ID applied to sources

devtool build recipe-name # check building 2.0.1 version

devtool deploy-target recipe-name root@${target_ip} # test the changes on target

devtool undeploy-target recipe-name root@${target_ip}

# delete the executable inside the "workspace/sources/recipe-name/"
# do the following command in build directory
devtool finish recipe-name ../meta-custom/ # after that command chech the recipe path in your meta-custom, you
# should see the recipe-name_2.0.1.bb instead of recipe-name.bb
# open the recipe-name_2.0.1.bb and check the changes 
```
<br>
<br>

----

**NOTE:** To make a kernel module recipe, you can watch this video: [Creating Kernel Module Recipe with Devtool](https://www.youtube.com/watch?v=LN_ulwXvE0k&list=PLwqS94HTEwpQmgL1UsSwNk_2tQdzq3eVJ&index=67)
