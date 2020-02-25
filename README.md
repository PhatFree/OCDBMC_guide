# OpenOCD in OpenBMC
This is a short guide on how to get started with OpenOCD in OpenBMC.
along with this guide, you can look at the arm remote debug refrence configuration image.

## Compiling into OpenBMC image

### adding OpenOCD into your image build conf.

An OpenOCD recipe already exists in Yocto/OpenEmbbded. to add openOCD into our image, all we really have to do is add `openocd` into our build components. one way to do that is add `IMAGE_INSTALL += "openocd"` into your bitbake `local.conf` build file.

**Current build environment:**
To add it into the current build environment, add `IMAGE_INSTALL += "openocd"` into your `../build/conf/local.conf`

**image default:** 
If if you'd like to add this to every build for the machine add it to the 
`<path_to_machine_conf>/local.conf.sample`
an example: `/meta-arm/meta-refrence/conf/local.conf.sample`

**arm reference:**
The arm reference has this done as an image default, so you'll just need to make sure you've used it as your conf, i.e.`export TEMPLATECONF=meta-arm/meta-reference/conf`

#### Congrats! This is all you need to have OpenOCD compiling into your image.
*Just to get openOCD compiled and running In OpenBMC you don't need to do any thing else.*
it will compile with the default config that supports FTDI adapters, and the upstreamed Targets (board/SOCs).

**To enable OpenOCD to work with a diffrent adapter, or if you need to apply patches to support new adpaters & targets, read the next secetion.**

---

## Adding adapters & targets

### updating the OpenOCD recipe with a bbappend

There's a good chance you'll want to add new adapters, change the compiled adapter, add new target files, startup scripts, etc.. If any of this sounds true, you'll need to create a BBappend file for OpenOCD

**An example file structure for bbappends with a patch:**

```
meta-arm
├── meta-reference
    ├── conf
    │   ├──...
    └── recipes-devtools
        └── openocd
            ├── openocd
            │   └── <patch-to-apply>.patch
            └── openocd_%.bbappend

```

**Adding patches:**
if you're adding new configs, or extending OpenOCD to work with a new JTAG adapter, the most likely way you'd do that is be applying patches to OpenOCD.
To add patches with new adapters, more targets, etc.. you can follow bitbake conventions, adding a bbappend `openocd_%.bbappend` then adding the patch to the source URI `SRC_URI += "file://<patch-to-apply>.patch"` like shown above.


**changing compile options (adpaters to support):**
the deafult recipe is loaced in `meta-openembedded/meta-oe/recipes-devtools/openocd/openocd_git.bb` and will only comiple with support for FTDI devices. 

In openocd you select your adpater, and give compilation flags before you compile, usally with `./configure` and your flags. in the yocto reciepe (`openocd_git.bb`) the defualt comiple options are here:`EXTRA_OECONF = "--enable-ftdi --disable-doxygen-html --disable-werror"`
This compiles for FTDI adpaters, disables building the devlopment doxygen docs, and turns off the gcc werror flag.


If we want to add or change adapters, we really need to change the compilation flags. we can do this by modifying `EXTRA_OECONF`  within our `openocd_%.bbappend`
 `--enable-<adpater>` based on what jtag adapter you have.

---

## Compiling & Running 

You should be able to compile your image, and install it on your BMC like normal. 

with the arm refrence, there's a patch that includeds OpenOCD/tcl script to simpfly running openocd with our Juno target, that can be run with `openocd -f target/arm/juno_RD_example.cfg`
This config enables debugging over networking, and setups openOCD with a BusBlaster and JunoR2.

alteritvley you can can invoke openocd like on a normal devlopment machine, with seprate configs for your ocd varbles (transport selection, network exposure etc..) and your adpater and target.
**example:**
`root@witherspoon:~# openocd -f openocd.cfg -f scripts/interface/dummy.cfg -f scripts/board/arm_evaluator7t.cfg`