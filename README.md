**2020/02/26** Known issues: 1) Won't compile MarkovBrain or ANNBrain on Visual Studio until the code is updated (should be soon) 2) None of MABE_extras have been updated to work with the latest build system because it has yet to be released on the MABE public branch, but is included here in this version of the animat-MABE codebase. I expect this to be released within a week or two.

#### Quickstart
If you kind of know what you're doing, here's a reminder of what's involved (assuming you have the right software installed - covered in Section 1):
```bash
git clone https://gitlab.msu.edu/neuro_animat/animatcpp
cd animatcpp
tools/setup.cmd #(on windows just: tools\setup)
./mbuild #(on windows: mbuild.exe)
cd work
## Now can run any of the following:
./mabe #(basic running mabe)
./mabe -s #(saves config files - see 'suggested configuration values' below)
./mabe -f *.cfg #(now using config files)
```

* **Suggested Configuration Values**
* `settings_world.cfg`
    * `WORLD_MOTORS2`
        * evluationsPerGeneration = 5
        * numThreads = < something small but non-zero if you don't want to hog an entire node on the HPCC, or your laptop >
* `settings.cfg`
    * `GLOBAL`
        * updates = < bigger number than 100 for anything meaningful >
    * `ARCHIVIST_DEFAULT`
        * realtimeSequence < your choice>
        * snapshotDataSequence < your choice>
        * snapshotOrganismsSequence < your choice>
        * writeSnapshotDataFiles = 1
        * writeSnapshotOrganismsFiles = 1
    * `ARCHIVIST_LODWAP`
        * dataSequence = < your choice>
        * organismsSequence = < your choice>
        * terminateAfter = 0
        * elitismCount = 10 # this is how many offspring each elite member produces
        * elitismRange = 10 # this is how many elites to select

##### Run MABE
* **Note:** If on the HPCC, you must load the appropriate compilers and libraries before running mabe, every time you log in: `source hpcc/loadmodules.sh`
* When successful, compiling MABE results in a mabe executable in the `work/` directory. `cd` into `work` and try running it to see if it works. By default it will run 110 generations of a Test environment, which should fly by in 1 or 2 seconds.
*(Remember, running an executable on the command line in Windows:* `exename`*or*`exename.exe` *and on Linux or Mac or MSYS2:* `./exename` *)*

#### Frequently Asked Questions

**When do I have to recompile with mbuild?**
If you've changed source code *or* changed which modules are included/excluded in `modules.txt`

**When do I have to generate settings files (mabe -s) after building?**
Only if you've added or removed settings defined in the `.h` and `.cpp` files. However, if you are only adding settings, you can have MABE keep your old settings and merge with the new ones it knows about: `mabe -f *.cfg -s` means, load the config files and the values I've set, now save out the settings again.

**On the HPCC, do I need to do everything again every time?**
No, if you've already compiled MABE, you will still find it in your `work/` directory. But you must do these steps every time you log in:
* log into a dev node
* load the hpcc modules for compiling or running MABE `source hpcc/loadmodules.sh`

#### Overview

The Reimers Lab digital animat project is a digital study organism to investigate questions around neurodevelopment, lifetime learning, evolutionary genetics, and evolutionary pressures. The rest of this document describes how to get the code, build and run, and what the sections of code are that you may be interested to modify.

1) [**Software: Install, Build, and Run**](#1-software-install-build-and-run)
    * We will get you up and running with the code base and comfortable compiling the code, with a separate guide for each OS.
2) [**MABE Architecture and Configuration**](#2-mabe-architecture-and-configuration)
    * We go over how MABE works, the different code sections, and how the animat code fits in.
3) [**Animat Architecture**](#31-animat-architecture-motors2world)
    * We detail the various parts of the animat code itself, so you know roughly where things are to add or modify.
4) [**c++17 and Beyond**](#4-c17-and-beyond)
    * We go over some of the newer c++ syntax used in the animat code, and point out the special libraries the animat code uses.
5) [**Loading and Analysis**](#5-loading-and-analysis)
    * Shows basics of loading saved agents/populations and how to export behavior data and other data, then how to visualize the behavior.

### 1) Software: Install, Build, and Run

The digital animat code uses a software framework called MABE (Modular Agent-Based Evolution framework) to facilitate faster scientific investigation.

#### Jump to Setup Section
* [Windows](#windows-setup)
* [MacOSX](#mac-setup)
* [Linux](#linux-setup)
* [HPCC](#hpcc-setup)

#### Overview
The overview of steps to get MABE running are as follows:
* 1a) Install or check that you have a **c++ compiler** (special HPCC instructions - see below)
* 1b) Install or check that you have **CMake** (special HPCC instructions - see below)
* 1c) Windows only: Install Intel's MKL for fast math ops
* 2 ) **Download** the animat/MABE source code
* 3 ) **Run** the tools/setup.cmd script on command line
* 4 ) **Run** the generated mbuild program on command line

## Windows Setup

#### 1) (Windows) Required Software
* 1a) **A C++ compiler** (fully updated only!) such as Visual Studio, XCode, GCC, or Clang. Others may work, but these are guaranteed to work. See `Optional Software` below for more.
* 1b) **CMake** Install with all defaults from [cmake.org](https://cmake.org/download/) (get the ...x64.msi file).
* 1c) **Intel MKL** Register and download but install ONLY AFTER Visual Studio: [https://software.intel.com/en-us/mkl/choose-download/windows](https://software.intel.com/en-us/mkl/choose-download/windows)

#### (Windows) Optional Software
* Use the **MSYS2** terminal instead of the normal terminal. Go the extra mile to change your home directory to be the same as on Windows terminal for better quality of life. See the MABE wiki Steps 1 and 2 of the [MinGW and Terminal](https://github.com/hintzelab/mabe/wiki/Installation-and-getting-started-with-MABE#installing-only-mingw-and-terminal-on-windows) for how to install and set up MSYS2.
* **GCC** or Clang can be installed once you install MSYS2. GCC is faster than visual studio to compile.
* **Visual Studio Code** is a popular editor for source code, especiallly suggested if you don't install the full Visual Studio.
* **git** if you want to keep up to date with latest changes in the repository, or track your own changes, or share your changes.

#### 2) (Windows) Get the code
* The animat code is hosted on MSU's GitLab service [MSU GitLab NeurAnimat](https://gitlab.msu.edu/neuro_animat/animatcpp)
* Either:
    * git clone the repository *(suggested)*
    * download the zip of the repository *(easiest, but eliminates ability to record your own changes and share)*
```bash
git clone git@gitlab.msu.edu:neuro_animat/animatcpp
```

#### 3) (Windows) Run First Time Setup
*This step is optional, but highly suggested because it sets up the* `mbuild` *executable for you, which is an easier way to interface with MABE. The next sections will assume you are using this tool, or if not, that you are savvy with c++ build tools enough to figure it out.*
* **Windows**: On a terminal, run the **setup.bat** file in the `tools` dir of the repository. For example, `animatcpp\tools\setup.cmd`
* **Mac or Linux or MSYS2**: On a terminal, run the **setup** file in the repository. For example, `animatcpp/tools/setup.cmd`
*Note: You can run the setup file from anywhere and it will copy the code/Utilities/{OS}_build binary to the repo root and rename it to* `mbuild`

**Windows CMD / Terminal**
```bash
cd animatcpp
tools\setup
```
**Windows MSYS2 Terminal**
```bash
cd animatcpp
sh tools/setup.cmd
```

#### 4) (Windows) Compile MABE
* Run `mbuild` inside the repository.
    * *If you've never run things on the command line before:*
    * `cd animatcpp` *then* `mbuild` *or* `mbuild.exe` *will both work*

**Windows CMD / Terminal**
```bash
mbuild
```
**MSYS2 Terminal**
```bash
./mbuild
```

-------
Finished! Now proceed with the `.cfg` file configuration and general running instructions at the top of this wiki.
-------




## Mac Setup

#### 1) (Mac) Required Software
* 1a) **A C++ compiler** (fully updated only!) such as Visual Studio, XCode, GCC, or Clang. Others may work, but these are guaranteed to work. See `Optional Software` below for more.
* 1b) **CMake** We can't use the homebrew CMake - please uninstall it. Get cmake from [cmake.org](https://cmake.org/download/) (get the ...dmg file)

#### (Mac) Optional Software
* **XCode**:
    * You can use **xcode** to edit your code, but any text editor will do, please update it to the latest available from Apple, or install your own C++ compiler from homebrew if your computer is too old to be updated.
    * Once you have Xcode, install the build tools using the command line: `xcode-select --install`
* **Python 3.6+** This is useful if you want to use the `mq` job control tool (local computer or HPCC modes) or the `mgraph` visualization tool.  Both tools are located in the `tools/` folder. For more information about these, see the MABE wiki [entry on mq](https://github.com/Hintzelab/MABE/wiki/MQ), or the [entry on mgraph](https://github.com/Hintzelab/MABE/wiki/MGraph). mbuild documentation on the MABE wiki is out of date as mbuild is no longer a python tool, so refer only to this documentation for mbuild.

#### 2) (Mac) Get the code
* The animat code is hosted on MSU's GitLab service [MSU GitLab NeurAnimat](https://gitlab.msu.edu/neuro_animat/animatcpp)
* Either:
    * git clone the repository *(suggested)*
    * download the zip of the repository *(easiest, but eliminates ability to record your own changes and share)*
```bash
git clone git@gitlab.msu.edu:neuro_animat/animatcpp
```

#### 3) (Mac) Run First Time Setup
*This step is optional, but highly suggested because it sets up the* `mbuild` *executable for you, which is an easier way to interface with MABE. The next sections will assume you are using this tool, or if not, that you are savvy with c++ build tools enough to figure it out.*
* **Windows**: On a terminal, run the **setup.bat** file in the `tools` dir of the repository. For example, `animatcpp\tools\setup.cmd`
* **Mac or Linux or MSYS2**: On a terminal, run the **setup** file in the repository. For example, `animatcpp/tools/setup.cmd`
*Note: You can run the setup file from anywhere and it will copy the code/Utilities/{OS}_build binary to the repo root and rename it to* `mbuild`

```bash
cd animatcpp
tools/setup.cmd
```

#### 4) (Mac) Compile MABE
* Run `mbuild` inside the repository.
    * *If you've never run things on the command line before:*
    * `cd animatcpp` then `./mbuild` (the ./ means run a file that is in the current directory)

```bash
./mbuild
```

-------
Finished! Now proceed with the `.cfg` file configuration and general running instructions at the top of this wiki.
-------


## Linux Setup

#### 1) (Linux) Required Software
* 1a) **A C++ compiler** A modern GCC, Clang, or PGI, etc. should work as long as it supports c++17.
* 1b) **CMake** Ubuntu 18.04 CMake is the minimum required, so most people should have access to a new enough version. If not, it's not hard to build from source.

#### (Linux) Optional Software
* **Python 3.6+** This is useful if you want to use the `mq` job control tool (local computer or HPCC modes) or the `mgraph` visualization tool.  Both tools are located in the `tools/` folder. For more information about these, see the MABE wiki [entry on mq](https://github.com/Hintzelab/MABE/wiki/MQ), or the [entry on mgraph](https://github.com/Hintzelab/MABE/wiki/MGraph). mbuild documentation on the MABE wiki is out of date as mbuild is no longer a python tool, so refer only to this documentation for mbuild.

#### 2) (Linux) Get the code
* The animat code is hosted on MSU's GitLab service [MSU GitLab NeurAnimat](https://gitlab.msu.edu/neuro_animat/animatcpp)
* Either:
    * git clone the repository *(suggested)*
    * download the zip of the repository *(easiest, but eliminates ability to record your own changes and share)*
```bash
git clone git@gitlab.msu.edu:neuro_animat/animatcpp
```

#### 3) (Linux) Run First Time Setup
*This step is optional, but highly suggested because it sets up the* `mbuild` *executable for you, which is an easier way to interface with MABE. The next sections will assume you are using this tool, or if not, that you are savvy with c++ build tools enough to figure it out.*
* On a terminal, run the **setup** file in the repository. For example, `animatcpp/tools/setup.cmd`
*Note: You can run the setup file from anywhere and it will copy the code/Utilities/{OS}_build binary to the repo root and rename it to* `mbuild`

```bash
cd animatcpp
tools/setup.cmd
```

#### 4) (Linux) Compile MABE
* Run `mbuild` inside the repository.
    * *If you've never run things on the command line before:*
    * `cd animatcpp` then `./mbuild` (the ./ means run a file that is in the current directory)

```bash
./mbuild
```

-------
Finished! Now proceed with the `.cfg` file configuration and general running instructions at the top of this wiki.
-------



## HPCC Setup

#### 1) (HPCC) Required Software
* Always, log into a dev node. Do not try to compile or run software on the Gateway node, or you will get strange errors.
The HPCC doesn't allow you to install software, but instead provides many different versions and installations you can 'activate'. All required software can be activated using the included `hpcc/loadmodules.sh` in a later step.

##### (HPCC) Optional Software
Skip this unless you are interested in alternative c++ compilers. It's possible to use specialty c++ compilers, such as PGI, or CUDA if you need that functionality. See `./mbuild -h` once you have everything working to see about changing compilers.

#### 2) (HPCC) Get the code
* The animat code is hosted on MSU's GitLab service [MSU GitLab NeurAnimat](https://gitlab.msu.edu/neuro_animat/animatcpp)
* Either:
    * git clone the repository *(suggested)*
    * download the zip of the repository *(easiest, but eliminates ability to record your own changes and share)*
```bash
git clone git@gitlab.msu.edu:neuro_animat/animatcpp
```

#### 3) (HPCC) Run First Time Setup
* Load required software for running or compiling MABE: `source hpcc/loadmodules.sh` (this will be required every time you log in)
* *This step is optional, but highly suggested because it sets up the* `mbuild` *executable for you, which is an easier way to interface with MABE. The next sections will assume you are using this tool, or if not, that you are savvy with c++ build tools enough to figure it out.*
* On a terminal, run the **setup** file in the repository. For example, `animatcpp/tools/setup.cmd`
*Note: You can run the setup file from anywhere and it will copy the code/Utilities/{OS}_build binary to the repo root and rename it to* `mbuild`

```bash
cd animatcpp
tools/setup.cmd
```

#### 4) (HPCC) Compile MABE
* Load required software for running or compiling MABE (if you haven't already): `source hpcc/loadmodules.sh` (this will be required every time you log in)
* Run `mbuild` inside the repository.
    * *If you've never run things on the command line before:*
    * `cd animatcpp` then `./mbuild` (the ./ means run a file that is in the current directory)

```bash
./mbuild
```

-------
Finished! Now proceed with the `.cfg` file configuration and general running instructions at the top of this wiki.
See the section on 'Running on the HPCC' below for information on submitting jobs.
-------

#### Running on the HPCC

There is a lot to know to run things on the HPCC! The `mq` tool helps with this (in the `tools/` dir). `mq` manages job submission of replicates and condition combinations, as well as allowing you to make use of the DMTCP system for Distributed MultiThreaded CheckPointing, which allows us to run jobs that will:

1) have high priority in the queue by claiming to run less than 4 hours
2) save state at 3hr 55min runtime, and quit, then resubmit for another 4 hour window, repeating until done.

The complexity of `mq` is in the condition, replication, and other settings configuration. This is done through a text file called `mq_conditions.txt`. A default template you should copy to your `work/` directory is in `tools/mq_conditions.txt`. At a high level, you can set the number of replicates to run (repeats of a stochastic observation), the MABE parameters you will be changing between conditions, and what values for those parameters you want to explore. To learn more about this, see the [MABE wiki entry](https://github.com/Hintzelab/MABE/wiki/MQ). In the future this template will be generated by the mq tool and will not exist as a separate file until generated on command.

Once you have configured your experimental conditions in your `work/` copy of `mq_conditions.txt`, then you can run them using the `mq` tool. There are several ways to run your experiment. Note you should edit the HPCC parameters at the bottom of the file to suit your job specifications. These are the same parameters as for any HPCC job submission - see iCER wiki or ask their staff for more information.

* `python ../tools/mq.py -h`: lists the help information, some of which are listed here
* `python ../tools/mq.py -l`: run your full experiment on your local computer
* `python ../tools/mq.py -n`: make the empty directories, and report would _would_ be run, but don't actually run.
* `python ../tools/mq.py -t`: test run. Tries to force very few replicates and very small number of generations, but runs all conditions. Good for testing if all the right input and output files look and work okay.
* `python ../tools/mq.py -d`: submit your full experiment to the HPCC, asking for the job run-time window you specified in `mq_conditions.txt` Must be running this command on the HPCC for it to work.
* `python ../tools/mq.py -i`: run in indefinite mode (recommended). This mode runs your job in 4 hr chunks, saving progress at the end of each chunk. Terminates when your request number of generations is reached. The reason this is good is the HPCC queueuing system prioritizes jobs that run in 4 hours or less and lets those jobs run on a wider number of computers. Must be running this command on the HPCC for it to work.

There are many HPCC-specific [job specifications](https://wiki.hpcc.msu.edu/display/ITH/List+of+Job+Specifications) and adjustments you may need to make (see the bottom of `mq_conditions.txt`), and the ICER's HPCC wiki and support open office hours are great ways to get help with this topic. See the HPCC's [Cluster Resources](https://wiki.hpcc.msu.edu/pages/viewpage.action?pageId=20120131) page to see what kinds of nodes are available. Intel18 has 20 cores (40 fake/hyperthreaded).



##### Advanced Compilation
Once you're familiar with building MABE, you may later want to explore altering the build process to exclude or include other modules. This is achieved by modifying the resulting **modules.txt** file, a few entries of which look like this, but the default will be to build the animat 2-Motors configuration:

```
% World
  - Logic16
  - Motors2
  * Test

% Genome
  * Circular
  - Multi

% Brain
  - Wire
  * CGP
```

The `*` ,`-` , and `+` symbols mean:
`*` excluded from compile
`+` included in compile
`-` included in compile and set as default for category

You can change these symbols in `modules.txt` and rebuild with `mbuild` to enable or disable the ability to select among optional modules at runtime. For instance, you might want to only ever compile with two different worlds and other worlds disabled, because you will be comparing them in your workflow and also don't need all the other modules or want to wait for them to compile.

The **mbuild** tool provides many options to simplify and alter the compilation of MABE. By default, it will compile whatever is defined in `modules.txt` in `Release` mode (meaning all performance optimizations enabled). However, you can change this to `debug` using the included `--debug` switch. To see all of the things you can do with `mbuild`, use the `-h` or `--help` flag.

**mbuild** can help you by automating:
* Building MABE
    * Using `modules.txt`
    * Using different compilers (such as NVidia GPU, or PGI)
    * In debug or release modes (release is default)
* Generating a project file to use with your favorite IDE (XCode, Visual Studio, etc.)
* Updating `modules.txt` based on the modules that exist in the MABE `code/` folder
* Making a new module from scratch
* Making a new module as a copy of another one
* Downloading new modules from the MABE_extras repository (these are being updated)

**Note** for Windows-MSYS2 users: colored display won't show up properly unless you always run by prepending `winpty`: `winpty ./mbuild`, but this is not necessary. You can install `winpty` through the package manager MSYS2 tool: `pacman -Ss winpty`



### 2) MABE Architecture and Configuration

##### Overview
MABE is an evolution framework, so by default it allows for selection, mutation, and inheritance, in an agent-based context, meaning each agent is evaluated in a way as if it has agency. MABE is comprised of 3 parts of immediate interest called *modules*: Worlds, Brains, and the rest of MABE, which has other modules. *(To read a more detailed overview, see [the MABE wiki](https://www.github.com/hintzelab/mabe/wiki))* Most importantly, MABE modules are truly modular, in that you can delete, copy, or share a module simply by moving its folder, and MABE will reconfigure itself based on your change the next time you go to build MABE.

* **Brain** is a any computational neural substrate capable of up to a full proprioceptive loop. Key concepts for a Brain module are Inputs, Updates, and Outputs.
    * Examples:
        * Aritificial Neural Network takes Inputs, allowed to Update some number of times, and provides Outputs.
        * Decision Tree Classifier takes Inputs, given 1 Update, and provides Outputs.
        * Game Theory, Prisoner's Dilemma Player takes no input, is given 1 Update, provides 1 Output.
* **World** is any evaluation method of quantitatively measuring performance of individual agents of a population. Key concepts for a World are the Population it's operating on, and the Scores the World is responsible to assign each agent.
    * Examples:
        * Foraging, each agent is iteratively: given inputs based on their location, allowed to update, and their outputs determine movement. Score is assigned based on food collected.
        * Classification, each agent is provided with a serial corpus of input patterns, given 1 update after each pattern, and scored based on correct output patterns.
        * Basic Game Theory, each agent is played against a specific strategy set, during which it is given no input, 1 update, and its output and opponent strategy pairing determine its score.
* **The Rest** The rest of MABE performs Selection, Inheritance, and Mutation among the population(s), as well as keeping track of lineage, performance metrics such as Score, and recording all these data to file. Additionally, MABE handles the technical bits of allowing settings to be configured through text files, and saving and reloading agents or entire populations. Lastly, the Brains and Worlds are modular to facilitate swapping them out and recombining them for new experiments. This is not always possible, as some brains will only work with some worlds, as is the case with the first one in this animat project, but this restriction is not symmetric and the animat world  module was designed to be paired with any brain module.

##### File Layout
MABE source code directory structure (in `code/`) is representative of the module categories. The important ones are:
- Archivists
- Brains
- Genomes
- Optimizers
- Worlds



* **Archivists** control how data is collected and saved. Different archivist provide different data views over evolutionary time, such as Line of Descent, or population Snapshots useful to studying speciation.
* **Brains** provide different ways to perform computations: input to output, such as the different between various neural network models.
* **Genomes** provide the underlying genetic structure, such as haploid, diploid, or triploid, and how mutations and other genetic events occur on those structures.
* **Optimizers** offer different ways to perform selection, such as elite selection, roulette selection, or lek selection.
* **Worlds** are the environments in which agents are evaluated and scored. The scores are what selection acts on.

#### Brain Module
As a biological analogy, every Brain module is essentially the part of the nervous system that has little or no concept of overall morphology (body design). With this understanding, the Brian is only responsible for doing 3 things: allowing its inputs to be set, performing a computational update on command, and setting outputs as a result of the update. Crucially, a MABE Brain is usually designed to be parameterized for any number of inputs and outputs. This is so when an experimenter decides to use a Brain X with some other World Y, that the World notifies the Brain at time of construction how many inputs and outputs it will need to process information for that World. The first animat brain (called `Motors2Brain`) will only ever use a specific amount of inputs and outputs, and for this reason it is limited to the accompanying animat world (called `Motors2World`).

Here are some of the most important functions:

* `<BrainName>(num_inputs, num_outputs, parameters_table)`
The name will be the name of the Brian, for instance, ANNBrain's constructor is `ANNBrain`. This is what the World will use to construct a Brain to use, and this is how it tells the Brain how many inputs and outputs it needs to operate in that World. We'll talk about parameters table in a later section.
* `resetBrain() -> void`
Provides a way for the World to indicate a contextual break, such as starting the evaluation over again to collect another stochastic data point. It's basically saying "hey Brain, what follows after this will not be related to what you've experienced so far, so forget all that"
* `setInput(index, value) -> void`
Provides a way that the World can set various input values for the Brain to process later when its `update()` function is called.
* `update() -> void`
Provides a way the World can let the brain do 1 whole computation or network update, which may result in the outputs being set.
* `readOutput(index) -> double`
Provides a way for the World to read the outputs that resulted from call to `update()`.

#### World Module
In biological analogy, a World module represents a task or environment for which agents may have their performance assessed. Typically, conceptually enticing Worlds are spatial, such as a foraging task where an agent is required to collect food. But Worlds may represent other performance assessments such as simply counting the number of a certain kind of gene in the genome. In this way Worlds allow a researcher to explore anything from digital animal behavior, to genetic computation.

Here are some of the most important functions and structures:

* `<WorldName>(parameters_table)`
The name will be the name of the World, for instance, TestWorld's constructor is `TestWorld`. This is usually called only once per MABE run, so it's a good place to set stuff up at the start of a simulation.
* `requiredGroups() -> mapping`
This function is called by MABE when MABE is setting up the initial populations and needs to know for the World you're using what the population structure needs to be and what the required number of inputs and outputs is for the Brains to run correctly in the World. Some worlds leave this as an optional parameter, and some worlds hard-code the required input/output numbers. It's called 'groups' because a World may specify that it needs more than one population.
* `evaluate(groups) -> void`
Once the World is constructed and all Brains are properly constructed to work with the World, then MABE will enter the evolution loop and iteratively call the `evaluate()` function. This function is responsible for assessing each agent and assign it a "Score" value, which is recorded into the **DataMap**.
* `DataMap`
DataMap is the way MABE records data and passes it around. Each organism in the population has its own DataMap and this is where you record score. DataMap allows you to set singular values (`set("score", value)`)  like if you know your World only evaluates each organism once you could use `set()`. DataMap also allows you to append in a list-like fashion (`append("score",value)`) like if you know your World may evaluate an organism 5 or more times, and you want MABE to average those scores when performing selection. `set()` and `append()` are also how you can store arbitrary data on a per-organism basis, more in the section on **Saving Data; Adding Parameters**.

#### Settings (Configuration) Files
See the excellent [MABE wiki entry for working with settings files](https://github.com/Hintzelab/MABE/wiki/Installation-and-getting-started-with-MABE#generating-settings-files).

#### Saving; Adding Parameters
* **Output Data .csv Files**
MABE generates a number of output `.csv` data files. To understand more about these, see the [MABE wiki entry](https://github.com/Hintzelab/MABE/wiki/Output-Files).
* **Adding Parameters**
You can add your own parameters to your modules, so that the values can be set via the text config files or command line parameters at runtime, and you don't need to recompile. See the [MABE wiki entry](https://github.com/Hintzelab/MABE/wiki/Adding-Parameters) for more information.
* **Saving Data (DataMap)**
`popFileColumns` is a vectors of strings that genomes, brains, and worlds each define for themselves. These vectors are used by the default archivist to determine what data from an organism's [DataMap](<https://github.com/Hintzelab/MABE/wiki/DataMap>) to include in average files (ex: pop.csv). To record your own novel data, such as `food_eaten`, you must first register it in the module's constructor by adding it to the `popFileColumns` vector:
```
popFileColumns.push_back("food_eaten");
```
Then, later during perhaps the end of an evaluation, you can set or append values to this new column:
```
org->dataMap.append("food_eaten", food_eaten_counter);
```
* **Saving Data (Manual)**
Alternatively, you may want to save very specific data and lots of it, perhaps for visualizing state later. You can use `FileManager.writeToFile(filename, datastring)`
The FileManager makes sure to blank the file at the start of MABE, and keep appending to that file every time you write to it.


### 3.1 Animat Architecture (Motors2World)

The Motors2 World, like any other world module, provides 2 important functions: `requiredGroups()`, and `evaluate()` that operates on an entire population (well, groups, but we just have one group). At time of writing, this is the only MABE module that is multithreaded, able to use as many cores as the host machine has. To facilitate this, an extra function was added that `evaluate()` uses called `evaluate_single_thread()`, which is used as the entry-point for each new thread created. That function evaluates a single agent, then loops again if there are more agents to evaluate, otherwise the thread exits. But, it is also responsible for orchestrating the evaluation itself, so this is the function with the lifetime loop. The basic structure is as follows, and you might notice something odd:

* `evaluate(population)`
    * `evaluate_single_thread() [first thread]`
    * `evaluate_single_thread() [second thread]`
    * `evaluate_single_thread() [third thread...]
        * `(multiple evaluations loop)`
            * `world.reset()`
            * `brain->resetBrain()`
            * `(lifetime loop)`	
                * `brain->update()`
                * `world.movePrey()`
                * `world.getInputSmell()`
                * `world.getOutputAndMove()`
                * `world.eat()`
            * `(record score)`

The odd thing you might realize is that some of these functions, such as `eat()` seem like they should instead belong to the agent. But the way we've separated concerns between Brain and World means anything to do with the World, involving location or quality of food, location of sensors, hunger levels of the body, etc. are all governed by the World. To achieve this information passing, `animat_hunger` and `calories` are both given as inputs to the brain. Where these values end up inside the Motors2Brain is the same as before porting to c++, but this passing-as-input allows separation of concerns and modularity so that other brains may be used with this World (such as a Brain version with different plasticity). However, it may have an odd effect when other brains are used with the Motors2World.

One final architectural note is that MABE Worlds typically have state variables such as animat position as member variables of the class instance. Because MABE only generally makes one instance of the World, then that means all animats share that same position information. This is not usually a problem, because before now, agents in MABE Worlds were not evaluated in parallel. For Motors2World to evaluate agents in parallel, each agent needed its own data structure of world variables, like hunger, position, number of prey, etc. To do this, there is now the Experience data structure that encapsulates all of these state variables, and a list of Experiences, identical to the list of Agents that make up the population. So agent 5 has the 5th experience structure.

### 3.2 Animat Architecture (Motors2Brain)

This version of Motors2Brain is direct-encoded in the MABE sense, meaning the genome is fixed to a certain size, and we know what each number in the genome does. This first version has 21 sites.

To facilitate direct-encding, this Brain tells the rest of MABE through its `requiredGenomes()` function that it needs no genome, and will handle such things itself. A side-benefit is that this is computationally faster. We also needed to override the following functions to deal with our local copy of the genome and appropriate mutations: `makeCopy()`, `makeBrain()`, `mutate()`, `serialize()`, `deserialize()`.

* `makeCopy()` function is used since we typically will use elitism of some form, which knows it should only copy, and leave mutation to us.
* `makeBrain()` function is called when a parent needs to reproduce, and we will eventually be relying only on this function instead of `makeCopy` to simplify things, but the Optimizer needs to be changed to make this work.
* `mutate()` only handles how to perturb the genetic values. Currently we take into account the known ranges and magnitude of changes for each of the 21 sites.
* `serialize()` and `deserialize()` are used by MABE to save and reload the data from file, enabling many other nice features. These functions are fairly straightforward.

The main `update()` function flows as follows, with code broken into logical functions as follows:

* `(neural refinement, once, if first time update called)`
    * `agent.spontaneousActivity()`
    * `agent.hebb()`
* `agent.updateNetwork()`
* `agent.updateNeuroModulators()`
* `agent.hebb()`
* `agent.sensoryAdaptation()`
* `agent.plasticity()`

Because the World will always call `update()` every time tick, then the Brain keeps track of the passing of time, and decides when to call the various functions that shouldn't happen every time tick, such as the function for homeostatic plasticity.

### 4) C++17 and Beyond

This animat code uses some advanced c++, which thankfully actually makes c++ easier! Here I'll describe a bit about how I'm using it if you've never seen it before.

**Trailing return types**
```c++
auto requiredGenomes() -> std::unordered_set<std::string> override {
```
This allows 2 things:
1) Visually aligns function names to always be at line position 6, instead of jumping around the page as you read down.
2) Allows the compiler to be smarter about some things.

**Designated initializers**
```c++
struct HebbConfig {
bool reinforce_always {false};
};
auto hebb(const HebbConfig& /*cfg*/) -> void;
// then called in other code later...
hebb({ .reinforce_always=true });
```
This allows 1 huge benefit: Named variables at the call site, so you can read the intent of the function call.

**Preincrement and Postincrement**
```c++
// fictional code
genome[++i] = 0;
genome[i++] = 0;
```
Okay, so not really a new thing, but good to remember since I use it sometimes.
* `++i` is preincrement, that is, increment `i`, *then* use the resulting value of `i` in the line of code.
* `i++` is postincrement, that is, use the value of `i` in the line of code, *then* increment `i`.

**PPrint**
```c++
bool not_python = true;
print("This is ",not_python);
```
The animat code uses and includes the pprint library, which improves quality of life when needing to print things to the screen. The alternative using standard c++ looks like this.
```c++
std::cout << "This is ", << not_python << std::endl;
```

**Armadilo math library**
To achieve speed while maintaining a somewhat familiar MATLAB-like workflow, this animat code is using the Armadillo Linear Algebra library. They have a very helpful [documentation page](http://arma.sourceforge.net/docs.html) that is easily searchable, but I found most things worked just like MATLAB, with only minor c++-ifications due to syntax restrictions.

### 5) Loading and Analysis

MABE has a feature to load 1 or more agents from the saved `*_organisms_*.csv` files that are saved either from population shapshot data, or line of descent data.
This feature is used through a configuration file called a `population_loader.plf` file. Here is the workflow:

* Generate template population loader file `./mabe -l` (that's a lowercase 'L')
* Edit this file to specify what you want to load and from where. The default file is full of commented out usage examples.
* For instance to load the most fit organism from a late-evolved population: `MASTER = greatest 1 by score_AVE from 'C0__MY_CONDITION_0/101/snapshot_organisms_900.csv'`
* Be sure to comment out the default `MASTER = default 100` line
* Use your population loader configuration by setting the population size to be the filename of this config file, either in the settings.cfg or on the command line:
 
```bash
./mabe -p GLOBAL-initPop population_loader.plf
```

If you are saving data, such as behavior data, make sure you are running with only one thread (multithreaded doesn't make sense here).
You will see in the `Motors2World.cpp` there is an if statement / config parameter controlling whether or not behavior data is written to file. That parameter is by convention the `mode` run mode of MABE (`./mabe -p GLOBAL-mode`) and by default the value is `run` but if you set it to `visualize` then the behavior data export will be triggered. You should use this pattern when exporting data, and you can even trigger on using the `analyze` mode.

#### Visualizing Behavior (or anything else)

Motors2World currently responds to the `GLOBAL-mode` setting if set to `visulualize` it will run the first organism it comes across, and save all lifetime behavior data to a file then quit.

I wrote a generic visualization tool, kind of like Processing, with example code used for animat visualization (2D-foraging) called [vis (download here)](https://github.com/joryschossau/vis).
To use this tool, copy or download the following files in that repository to your `work/` dir: `releases/pck/pck/vis.pck`, `releases/exe/<your_os>/vis`, and for the animat behavior script I already wrote get this script `examples/2d-foraging/readcsv.gd`.
Then, use it like this:

```bash
./vis --2d --script=readcsv.gd --file=animat_data.csv
```

* `animat_data.csv` is the resulting file from MABE when running Motors2World in Visualize mode. It has the positio/rotation of the agent and prey information from one lifetime of the first organism in the population.
* `./vis`: is the visualization tool
* `--script=readcsv.gd`: tells the visualization tool to run the `readcsv.gd`
* `--file=animat_data.csv`: the `readcsv.gd` script tells `vis` that it needs a command line parameter `--file` for what position data to visualize.

While you can use it purely on the command line and with any text editor, it's most enjoyable to use with the full IDE experience with code error detection and autocompletion. To do that you should download the **Godot** game engine (no installation required), import a project, and use the `src/` directory from the `vis` repository as a project directory. After that, since you are not using the command line, you'll need to set the default command line arguments that `vis` expects, under `settings` menu, `project settings`, `General` tab, `Editor` category, and set `Main Run Args` to `--2d --script=readcsv.gd --file=animat_behavior.csv`. The `readcsv.gd` file is included in the vis repo in the examples/demos, you can use that. The `animat_behavior.csv` is whatever `.csv` file is generated by the animat `Motors2World` code.

The `vis` tool expects either `--2d` or `--3d`, then the `--script=` paramter should point to your visualization script. The visualization script `readcsv.gd` instructs `vis` to expect another parameter, called `--file`, which is a csv file containing the data it will plot and animate.