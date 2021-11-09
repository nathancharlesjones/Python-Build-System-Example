# Example Project using Python Build System

## What is it?

A C project that demonstrates use of the [Python Build System](https://github.com/nathancharlesjones/Python-Build-System).

## How do I use it?

1) Clone this repo; use `git clone --recurse-submodules https://github.com/nathancharlesjones/Python-Build-System-Example` to ensure that you also get the `Python-Build-System` subfolder.

2) If you're planning on using the Dockerfile:
    - Inspect the Dockerfile to see if there are any additional programs you'll want. Only `build-essentials` is required for normal GCC projects (it includes `gcc`, `g++`, and `make`).
    - Download and install [Docker](https://docs.docker.com/get-docker/). Ensure it is running.
    - From a shell on your system, navigate to `./Python-Build-System` and run the following, where `<NAME>` is the name you want to give your Docker image:

    `docker build -f Dockerfile -t <NAME> .`
    
    - Wait. Building this Docker image takes a good 5-10 minutes on my system.

3) From a shell on your system, navigate back to this project's root folder. Run the following to build the project:

`./Python-Build-System/make.py -b`

---
**NOTE**

The filepaths used in these Python files are only really set up for a Unix system. If you're trying to build a project with this script on Windows, you'll need to use Docker/WSL, or rewrite the parts of the files that use Unix-specific conventions, such as for filepaths or for shell commands like `rm`.

---

Or, if you're using Docker:

`docker run -it --rm -v ${PWD}:/app <NAME> /bin/bash -c "./Python-Build-System/make.py -b"`,

where `<NAME>` is the same name you gave the Docker container above.

If you see an error like `bash: ./Simple-Build-System/make.py: /bin/python3^M: bad interpreter: No such file or directory` it's probably because you're editing `make.py` on Windows (and using Windows line endings, CRLF) but the file is being run on a Unix machine (which is expecting Unix line endings, LF only). If this is the problem, you'll need to figure out how to change to Unix line endings. The simplest fix seems to be to change the default line ending in your text editor; I use Sublime Text and [this thread](https://stackoverflow.com/questions/39680585/how-do-configure-sublime-to-always-convert-to-unix-line-endings-on-save) recommended I add the following keys to my user settings:
```
// Determines what character(s) are used to terminate each line in new files.
// Valid values are 'system' (whatever the OS uses), 'windows' (CRLF) and
// 'unix' (LF only).
"default_line_ending": "unix",
// Display file encoding in the status bar
"show_encoding": true,
// Display line endings in the status bar
"show_line_endings": true
```
Once I did, I could select the line ending I wanted on the bottom toolbar in Sublime Text. Many forums I read when trying to solve this problem also recommended a program called dos2unix.

4) Run `./build/x86/debug/blinky_x86_debug.exe` (or `docker run -it --rm -v ${PWD}:/app <NAME> /bin/bash -c "./build/x86/debug/blinky_x86_debug.exe"`) to see the program running on your host machine. Load the STM32 version of the program (located at `./build/STM32F1/debug/blinky_STM32F1_debug.bin`) onto an STM32F1 device using the debug adapter of your choice to see an LED blinking on pin PC13.

## How does it work?

The Python Build System is a simple Python script that acts like GNU `make`, but with a nicer interface: each library or executable is defined in `Python-Build-System/project_targets.py` in a simple dictionary format. The script `Python-Build-System/make.py` reads that file and then builds each target according to a simple set of rules. Changing how each target is built or adding or removing targets is as easy as editing the dictionary called `targets` in `project_targets.py`. Since the entire build system is written in Python, editing it to add extra command-line flags or to automate certain other parts of your project is as easy as rewriting the Python script. See the [project repo](https://github.com/nathancharlesjones/Python-Build-System) for additional details.

This project also demonstrates one way of writing a cross-platform project. It does this using a technique called ["link-time polymorphism"](https://github.com/nathancharlesjones/Comparison-of-OOP-techniques-in-C/tree/main/1c_Link-time-Polymorphism_ADT), in which a single header file (`hardware/include/hardwareAPI.h`) is given more than one implementation (`hardware/source/x86/x86.c` for the host machine and `hardware/source/STM32F1/source/STM32F1.c` for the STM32). To avoid multiple definition errors, only one of those source files is linked in with the final executable (notice the different set of source files listed in `project_targets.py` for each of our two targets), which is fine because when we compile the program we only _want_ a single definition for the hardware functions.

Notice how, in `source/main.c`, we need to implement a `delay_sec` function. This function is defined in `hardware/hardwareAPI.h` (and it would have a nice description if I was a better programmer), but we don't know, yet, how it's implemented. In `hardware/source/x86/x86.c`, `delay_sec` is implemented by using `struct timespec` and related functions from `time.h`, a Linux system header. And in `hardware/source/STM32F1/source/STM32F1.c` (a modified version of the `main.c` that was generated for this project from STM32CubeMX), `delay_sec` is implemented using the `HAL_Delay` function that the STM32 HAL provides us. Each one provides the needed functionality when used with the appropriate piece of hardware.
