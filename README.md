# Yocto Commands

---

## Setup yocto from scratch

-   This section explains how to create a workspace, download the Poky
    (Yocto) reference distribution, and prepare a build environment.

``` bash
# Create a new directory named 'yocto-sandbox'
mkdir yocto-sandbox

# Change the current directory to 'yocto-sandbox'
cd yocto-sandbox
```

-   Clone the Poky repository (Yocto Project reference distribution)
    with the 'scarthgap' branch:

``` bash
git clone git://git.yoctoproject.org/poky -b scarthgap src/poky
```

-   Setup the cqfd. Copy the `.cqfd` folder from this repo to your
    project.

``` bash
export CQFD_SHELL=/bin/bash
cqfd init
cqfd shell
source src/poky/oe-init-build-env
```

---

## Create your own custom layer

-   Use `create-layer` option to create the layer.

``` bash
bitbake-layers create-layer ../src/meta-playground
```

-   Initialize Git in Your New Layer

``` bash
git -C ../src/meta-playground/ init
git -C ../src/meta-playground/ add .
git -C ../src/meta-playground/ config user.name "abhishek ojha"
git -C ../src/meta-playground/ config user.email "abhishekojha38@gmail.com"
git -C ../src/meta-playground/ commit -m "initial commit"
```

-   Add Layer to bblayers.conf

``` bash
bitbake-layers show-layers
bitbake-layers add-layer ../src/meta-playground/
bitbake-layers show-layers
```

-   Configure Local Build for QEMU x86-64

``` bash
cat << EOF > conf/local.conf
INHERIT += "buildhistory"
MACHINE = "qemux86-64"
DISTRO = "poky-altcfg"
DISTRO_FEATURES = "systemd ext4"
EXTRA_IMAGE_FEATURES += "empty-root-password"
EOF
```

-   Build the Image

``` bash
bitbake core-image-minimal
```

---

## Save the layer

-   Generate Layer Setup Script

``` bash
bitbake-layers create-layers-setup ../src/meta-playground/
```

-   Save Build Configuration Template

``` bash
bitbake-layers save-build-conf ../src/meta-playground playground
```

This stores: \* local.conf \* bblayers.conf \* template metadata

-   Commit the Saved Template

``` bash
git -C ../src/meta-playground add .
git -C ../src/meta-playground commit -m "save setup-layers and template"
```

---

## How To Use Custom Layer (New Project)

-   Clone the Layer

``` bash
git clone <github.com>/meta-playground
# OR local copy
git clone ~/Practice/yocto/yocto-sandbox/src/meta-playground \
          ~/Practice/yocto/yocto-sandbox2/src/meta-playground
```

-   Copy CQFD Config Into New Project

``` bash
cd yocto-sandbox2/
cp .cqfd* . -rf
```

-   Initialize CQFD

``` bash
export CQFD_SHELL=/bin/bash
cqfd init
cqfd shell
```

-   Run Layer Setup Script

``` bash
./src/meta-playground/setup-layers
```

This restores: \* Layer paths \* Dependencies \* bblayers.conf content

-   Load Build Environment with Template

``` bash
TEMPLATECONF=$PWD/src/meta-playground/conf/templates/playground \
    source ./src/poky/oe-init-build-env build-x86
```

-   Build Image

``` bash
bitbake core-image-minimal
```

You now reproduce the exact same build in a clean project.

---

## Create new recipe

You can always write a recipe entirely from scratch. However, Yocto
provides three convenient options to help you get started more quickly:


**Using Existing Recipes** -- Locate an existing recipe with similar
functionality and modify it to fit your needs.

### Using `Devtool`
Assists in generating a new recipe and sets up a development-friendly workspace.

#### Creating the Base Recipe Using *devtool add*

To quickly create a new recipe in a Yocto development environment, the
preferred workflow is to use the `devtool` utility.

The following example demonstrates creating a recipe for the **nano**
text editor.

-   Create the Initial Recipe. Use the devtool add command to generate a
development recipe and fetch the source code.

``` bash
devtool add nano https://git.savannah.gnu.org/git/nano.git
```

-   Edit the Temporary Recipe. To modify the automatically generated recipe

``` bash
devtool edit-recipe nano
```

-   Build the Recipe in the Workspace. Build the software using the development
workspace recipe.

``` bash
devtool build nano
```

-   Finalize and Export the Recipe Into Your Layer. Once you confirm that the
build succeeds and the recipe works as expected, finish the recipe and move it
into your custom layer:

``` bash
# Save all the changes in ../src/meta-playground/recipe
devtool finish nano ../src/meta-playground/
```

- Output Layer Structure. Below is an example directory layout after devtool
finish writes the recipe into meta-playground:

```diagram
src/meta-playground/
├── conf
│   ├── layer.conf
│   └── templates
│       └── playground
├── COPYING.MIT
├── README
├── recipes-nano
│   └── nano
│       └── nano_git.bb
├── setup-layers
└── setup-layers.json
```

At this point, nano_git.bb is now a permanent part of your layer and can be
built like any other recipe

```bash
bitbake nano
```

This completes the process of creating a base recipe using devtool add.

#### Modify the Base Recipe Using *devtool modify*

- Yocto’s devtool modify workflow allows you to extract a recipe’s source code
into a workspace, make changes, build the package, test it on a running target,
and finally integrate it into your layer.

-   Modify the Recipe

``` bash
# `nano` recipe is extracted to `workspace/sources/nano/`.
devtool modify nano
```

The source of the nano recipe is extracted to workspace/sources/nano/.

-   Build the Recipe

``` bash
# do the chnages in `workspace/sources/nano/` folder and build
devtool build nano
```

-   Build a Complete Image

``` bash
# build the image with changes
devtool build-image <image-name>
```

-   Deploy the Recipe

``` bash
# deploy image on hardware or qemu
devtool deploy-target nano root@192.168.7.2
```

-   Finish the Recipe

``` bash
# Save all the changes in ../src/meta-playground/recipe
devtool finish nano ../src/meta-playground/
```

### Using `recipetool`

Automatically generates a base recipe from the provided source files.

#### Create the recipe using `recipetool`

```bash
# Create folder for recipe
$ mkdir -p ../sources/meta-playground/meta-playground-os/recipes-jsonc/json-c-demo/

# Create recipe
$ recipetool create https://github.com/json-c/json-c.git \
    -o ../sources/meta-playground/meta-playground-os/recipes-jsonc/json-c-demo/json-c-demo_git.bb
```

- `json-c-demo_git.bb` will be created inside `recipes-jsonc/json-c-demo` recipe.

```
# Recipe created by recipetool
# This is the basis of a recipe and may need further editing in order to be fully functional.
# (Feel free to remove these comments when editing.)

# WARNING: the following LICENSE and LIC_FILES_CHKSUM values are best guesses - it is
# your responsibility to verify that the values are complete and correct.
#
# The following license files were not able to be identified and are
# represented as "Unknown" below, you will need to check them yourself:
#   COPYING
LICENSE = "Unknown"
LIC_FILES_CHKSUM = "file://COPYING;md5=de54b60fbbc35123ba193fea8ee216f2"

SRC_URI = "git://github.com/json-c/json-c.git;protocol=https;branch=master"

# Modify these as desired
PV = "1.0+git"
SRCREV = "a1249bfda0f6adf0363d5ab42d9ca816b3366ff2"

S = "${WORKDIR}/git"

# NOTE: unable to map the following CMake package dependencies: json-c Doxygen
DEPENDS = "json-c"

inherit cmake pkgconfig

# Specify any options you want to pass to cmake using EXTRA_OECMAKE:
EXTRA_OECMAKE = ""
```

- Check is recipe is ready

```bash
$ bitbake-layers show-recipes | grep json-c-demo
json-c-demo:
```

- Build the json-c

```bash
bitbake json-c-demo
```

## References

-   https://docs.yoctoproject.org/ref-manual/index.html
