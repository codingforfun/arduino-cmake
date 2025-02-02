=============
Arduino CMake
=============

Arduino is a great development platform, which is easy to use. It has everything a beginner should need. The *Arduino IDE* simplifies a lot of things for the standard user, but if you are a professional programmer the IDE can feel simplistic and restrictive.

One major drawback of the *Arduino IDE* is that you cannot do anything without it, which for me is a **complete buzz kill**. Thats why I created an alternative build system for the Arduino using CMake.

CMake is great corss-platform build system that works on practically any operating system. With it you are not constrained to a single build system. CMake lets you generated the build system that fits your needs, using the tools you like. It can generate any type of build system, from simple Makefiles, to complete projects for Eclipse, Visual Studio, XCode, etc.

The **Arduino CMake** build system integrates tightly with the *Arduino SDK*. Version **1.0** of the *Arduino SDK* is required.

So if you like to do things from the command line (using make), or to build you're firmware where you're in control, or if you would like to use an IDE such as Eclipse, KDevelop, XCode, CodeBlocks or something similar,  then **Arduino CMake** is the system for you.

Features
--------

* Integrates with *Arduino SDK*
* Supports all Arduino boards.
* Supports Arduino type libraries
* Automatic detection of Arduino libraries.
* Generates firmware images.
* Generates libraries.
* Upload support.
* Programmer support (with bootloader upload).
* Supports multiple build system types (Makefiles, Eclipse, KDevelop, CodeBlocks, XCode, etc).
* Cross-platform: Windows, Linux, Mac
* Extensible build system, thanks to CMake


Feedback
--------

**Arduino CMake** is hosted on GitHUB and is available at:

https://github.com/queezythegreat/arduino-cmake

Did you find a bug or would like a specific feature, please report it at:

https://github.com/queezythegreat/arduino-cmake/issues

If you would like to hack on this project, don't hesitate to fork it on GitHub.
I will be glad to integrate you'r changes if you send me a ``Pull Request``.


Requirements
------------

* Base requirements:

  - ``CMake`` - http://www.cmake.org/cmake/resources/software.html
  - ``Arduino SDK`` - http://www.arduino.cc/en/Main/Software

* Linux requirements:

  - ``gcc-avr``      - AVR GNU GCC compiler
  - ``binutils-avr`` - AVR binary tools
  - ``avr-libc``     - AVR C library
  - ``avrdude``      - Firmware uploader


Contributors
------------

I would like to thank the following people for contributing to **Arduino CMake**:

* Marc Plano-Lesay (`Kernald`_)
* James Goppert (`jgoppert`_)
* Matt Tyler (`matt-tyler`_)

.. _Kernald: https://github.com/Kernald
.. _jgoppert: https://github.com/jgoppert
.. _matt-tyler: https://github.com/matt-tyler


TODO
----

* Sketch conversion (PDE files)
* Test more complex configurations and error handling

Contents
--------

1. `Getting Started`_
2. `Using Arduino CMake`_

   1. `Creating firmware images`_
   2. `Creating libraries`_
   3. `Arduino Libraries`_
   4. `Compiler and Linker Flags`_
   5. `Programmers`_

3. `Linux Environment Setup`_

   1. `Serial Namming`_
   2. `Serial Terminal`_

4. `Mac OS X Environment Setup`_

   1. `Serial Namming`_
   2. `Serial Terminal`_

5. `Windows Environment Setup`_

   1. `CMake Generators`_
   2. `Serial Namming`_
   3. `Serial Terminal`_

6. `Eclipse Environment`_
7. `Troubleshooting`_

   1. `undefined reference to `__cxa_pure_virtual'`_
   2. `Arduino Mega 2560 image does not work`_
   3. `Library not detected automatically`_

8. `Resources`_






Getting Started
---------------


The following instructions are for **\*nix** type systems, specifically this is a Linux example.

In short you can get up and running using the following commands::

    mkdir build
    cd build
    cmake ..
    make
    make upload              # to upload all firmware images             [optional]
    make wire_reader-serial  # to get a serial terminal to wire_serial   [optional]

For a more detailed explanation, please read on...

1. Toolchain file
   
   In order to build firmware for the Arduino you have to specify a toolchain file to enable cross-compilation. There are two ways of specifying the file, either at the command line or from within the *CMakeLists.txt* configuration files. The bundled example uses the second approach like so::

        set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/cmake/toolchains/Arduino.cmake)

   Please note that this must be before the ``project(...)`` command.
   
   If you would like to specify it from the command line, heres how::

        cmake -DCMAKE_TOOLCHAIN_FILE=../path/to/toolchain/file.cmake PATH_TO_SOURCE_DIR

2. Creating a build directory

   The second order of business is creating a build directory. CMake has a great feature called out-of-source builds, what this means is the building is done in a completely separate directory, than where the sources are. The benefits of this is you don't have any clutter in you source directory and you won't accidentally commit something in, that is auto-generated.

   So lets create that build directory::

        mkdir build
        cd build

3. Creating the build system

   Now lets create the build system that will create our firmware::

        cmake ..

   To specify the build system type, use the ``-G`` option, for example::

        cmake -G"Eclipse CDT4 - Unix Makefiles" ..

   If you rather use a GUI, use::

        cmake-gui ..

4. Building

   Next we will build everything::

        make

5. Uploading

   Once everything built correctly we can upload. Depending on your Arduino you will have to update the serial port used for uploading the firmware. To change the port please edit the following variable in *CMakeLists.txt*::

        set(${FIRMWARE_NAME}_PORT /path/to/device)

   Ok lets do a upload of all firmware images::

        make upload

6. Serial output

   If you have some serial output, you can launch a serial terminal from the build system. The command used for executing the serial terminal is user configurable by the following setting::

        set(${FIRMWARE_NAME}_SERIAL serial command goes here)

   In order to get access to the serial port use the following in your command::

        @INPUT_PORT@

   That constant will get replaced with the actual serial port used (see uploading). In the case of our example configuration we can get the serial terminal by executing the following::

        make wire_reader-serial










Using Arduino CMake
-------------------

The first step in generating Arduino firmware is including the **Arduino CMake** module package. This easily done with::

    find_package(Arduino)

To have a specific minimal version of the *Arduino SDK*, you can specify the version like so::

    find_package(Arduino 1.0)

That will require an *Arduino SDK* version **1.0** or newer. To ensure that the SDK is detected you can add the **REQUIRED** keyword::


    find_package(Arduino 1.0 REQUIRED)


Creating firmware images
~~~~~~~~~~~~~~~~~~~~~~~~

Once you have the **Arduino CMake** package loaded you can start defining firmware images.

To create Arduino firmware in CMake you use the ``generate_arduino_firmware`` command. This function only accepts a single argument, the target name. To configure the target you need to specify a list of variables of the following format before the command::

    ${TARGET_NAME}${OPTION_SUFFIX}

Where ``${TARGET_NAME}`` is the name of you target and ``${OPTIONS_SUFFIX}`` is one of the following option suffixes::

     _SRCS           # Target source files
     _HDRS           # Target Headers files (for project based build systems)
     _SKETCHES       # Target sketch files
     _LIBS           # Libraries to linked against target (sets up dependency tracking)
     _BOARD          # Board name (such as uno, mega2560, ...)
     _PORT           # Serial port, for upload and serial targets [OPTIONAL]
     _SERIAL         # Serial command for serial target           [OPTIONAL]
     _NO_AUTOLIBS    # Disables Arduino library detection (default On)
     _AFLAGS         # Overide global avrdude flags for target
     _PROGRAMMER     # Programmer name, enables programmer burning (including bootloader).


So to create a target (firmware image) called ``blink``, composed of ``blink.h`` and ``blink.cpp`` source files for the *Arduino Uno*, you write the following::

    set(blink_SRCS  blink.cpp)
    set(blink_HDRS  blink.h)
    set(blink_BOARD uno)

    generate_arduino_firmware(blink)

Upload Firmware
_______________

To enable firmware upload functionality, you need to add the ``_PORT`` settings::

    set(blink_PORT /dev/ttyUSB0)

Once defined there will be two targets available for uploading, ``${TARGET_NAME}-upload`` and a global ``upload`` target (which will depend on all other upload targets defined in the build):

* ``blink-upload`` - will upload just the ``blink`` firmware
* ``upload`` - upload all firmware images registered for uploading

Serial Terminal
_______________
To enable serial terminal, add the ``_SERIAL`` setting (``@INPUT_PORT@`` will be replaced with the ``blink_PORT`` setting)::

    set(blink_SERIAL picocom @INPUT_PORT@ -b 9600 -l)

This will create a target named ``${TARGET_NAME}-serial`` (in this example: blink-serial).




Creating libraries
~~~~~~~~~~~~~~~~~~

Creating libraries is very similar to defining a firmware image, except we use the ``generate_arduino_library`` command. The syntax of the settings is the same except we have a different list of settings::

     _SRCS           # Library Sources
     _HDRS           # Library Headers
     _LIBS           # Libraries to linked in (sets up dependency tracking)
     _BOARD          # Board name (such as uno, mega2560, ...)
     _NO_AUTOLIBS    # Disables Arduino library detection

Lets define a simple library called ``blink_lib``, with two sources files for the *Arduino Uno*::


    set(blink_lib_SRCS  blink_lib.cpp)
    set(blink_lib_HDRS  blink_lib.h)
    set(blink_lib_BOARD uno)

    generate_arduino_firmware(blink_lib)

Once that library is defined we can use it in our other firmware images... Lets add ``blink_lib`` to the ``blink`` firmware::

    set(blink_SRCS  blink.cpp)
    set(blink_HDRS  blink.h)
    set(blink_LIBS  blink_lib)
    set(blink_BOARD uno)

    generate_arduino_firmware(blink)

CMake has automatic dependency tracking, so when you build the ``blink`` target, ``blink_lib`` will automatically get built, in the right order.





Arduino Libraries
~~~~~~~~~~~~~~~~~

Libraries are one of the more powerful features which the Arduino offers to users. Instead of rewriting code, people bundle their code in libraries and share them with others.
The structure of these libraries is very simple, which makes them easy to create.

An Arduino library is **any directory which contains a header named after the directory**, simple.
Any source files contained within that directory is part of the library. Here is a example of library a called ExampleLib::

    ExampleLib/
      |-- ExampleLib.h
      |-- ExampleLib.cpp
      `-- OtherLibSource.cpp

Now because the power of Arduino lies within those user created libraries, support for them is built right into **Arduino CMake**. The **Arduino SDK** comes with a large number of default libraries, adding new libraries is simple.

To incorporate a library into your firmware, you can do one of three things:

1. Place the library next to the default Arduino libraries (located at **${ARDUINO_SDK}/libraries**)
2. Place the library next to the firmware configuration file (same directory as the **CMakeLists.txt**)
3. Place the library in a separate folder and tell **Arduino CMake** the path to that directory.
   
   To tell CMake where to search for libraries use the `link_directories` command. The command has to be used before defining any firmware or libraries requiring those libraries.
   
   For example::
     
      link_directories(${CMAKE_CURRENT_SOURCE_DIR}/libraries)
      link_directories(/home/username/arduino_libraries)


If a library contains nested sources, a special option must be defined to enable recursion. For example to enable recursion for the Arduino Wire library use::

    set(Wire_RECURSE True)

The option name should be **${LIBRARY_NAME}_RECURSE**, where in this case **LIBRARY_NAME** is equal to *Wire*.



Compiler and Linker Flags
~~~~~~~~~~~~~~~~~~~~~~~~~

The default compiler and linker flags should be fine for most projects. If you required specific compiler/linker flags, use the following options to change them:

* **ARDUINO_C_FLAGS** - C compiler flags
* **ARDUINO_CXX_FLAGS** - C++ compiler flags
* **ARDUINO_LINKER_FLAGS** - Linker flags


Set these option either before the `project()` like so::

    set(ARDUINO_C_FLAGS      "-ffunction-sections -fdata-sections")
    set(ARDUINO_CXX_FLAGS    "${ARDUINO_C_FLAGS} -fno-exceptions")
    set(ARDUINO_LINKER_FLAGS "-Wl,--gc-sections")
    project(ArduinoExample C CXX)

or when configuring the project::

    cmake -D"ARDUINO_C_FLAGS=-ffunction-sections -fdata-sections" ../path/to/sources/


Programmers
~~~~~~~~~~~

**Arduino CMake** fully supports programmers, for burning firmware and bootloader images directly onto the Arduino. 
If you have a programmer that is supported by the *Arduino SDK*, everything should work out of the box.
As of version 1.0 of the *Arduino SDK*, the following programmers are supported:

* **avrisp** - AVR ISP
* **avrispmkii** - AVRISP mkII
* **usbtinyisp** - USBtinyISP
* **parallel** - Parallel Programmer
* **arduinoisp** - Arduino as ISP

The programmers.txt file located in `${ARDUINO_SDK}/hardware/arduino/` lists all supported programmers by the *Arduino SDK*.

In order to enable programmer support, you have to define the following setting::

    set(${TARGET_NAME}_PROGRAMMER programmer_id)

where `programmer_id` is the name of the programmer supported by the *Arduino SDK*.

Once you have enabled programmer support, two new targets are available in the build system:

* **${TARGET_NAME}-burn** - burns the firmware image via the programmer
* **${TARGET_NAME}-burn-bootloader** - burns the original **Arduino bootloader** image via the programmer

If you need to restore the original **Arduino bootloader** onto your Arduino, so that you can use the traditional way of uploading firmware images via the bootloader, use **${TARGET_NAME}-burn-bootloader** to restore it.



Linux Environment Setup
-----------------------

Running the *Arduino SDK* on Linux is a little bit more involved, because not everything is bundled with the SDK. The AVR GCC toolchain is not distributed alongside the Arduino SDK, so it has to be installed seperately.

To get **Arduino CMake** up and running follow these steps:

1. Install the following packages using your package manager:
    
   * ``gcc-avr``      - AVR GNU GCC compiler
   * ``binutils-avr`` - AVR binary tools
   * ``avr-libc``     - AVR C library
   * ``avrdude``      - Firmware uploader
    
2. Install the *Arduino SDK*.
    
   Depending on your distribution, the *Arduino SDK* may or may not be available.
    
   If it is available please install it using your packages manager otherwise do:
    
   1. Download the `Arduino SDK`_
   2. Extract it into ``/usr/share``
    
   NOTE: Arduino version **1.0** or newer is required!

3. Install CMake:
    
   * Using the package manager or
   * Using the `CMake installer`_

   NOTE: CMake version 2.8 or newer is required!



Serial Naming
~~~~~~~~~~~~~

On Linux the Arduino serial device is named as follows (where **X** is the device number)::

    /dev/ttyUSBX
    /dev/ttyACMX

Where ``/dev/ttyACMX`` is for the new **Uno** and **Mega** Arduino's, while ``/dev/ttyUSBX`` is for the old ones.

CMake configuration example::

    set(${FIRMWARE_NAME}_PORT /dev/ttyUSB0)


Serial Terminal
~~~~~~~~~~~~~~~

On Linux a wide range on serial terminal are availabe. Here is a list of a couple:

* ``minicom``
* ``picocom``
* ``gtkterm``
* ``screen``











Mac OS X Environment Setup
-------------------------

The *Arduino SDK*, as on Windows, is self contained and has everything needed for building. To get started do the following:

1. Install the  *Arduino SDK*

   1. Download `Arduino SDK`_
   2. Copy ``Arduino`` into ``Applications``
   3. Install ``FTDIUSBSerialDrviver*`` (for FTDI USB Serial)

2. Install CMake
   
   1. Download `CMake`_
   2. Install ``cmake-*.pkg``
        
      NOTE: Make sure to click on **`Install Command Line Links`**

Serial Naming
~~~~~~~~~~~~~

When specifying the serial port name on Mac OS X, use the following names (where XXX is a unique ID)::

    /dev/tty.usbmodemXXX
    /dev/tty.usbserialXXX

Where ``tty.usbmodemXXX`` is for new **Uno** and **Mega** Arduino's, while ``tty.usbserialXXX`` are the older ones. 

CMake configuration example::

    set(${FIRMWARE_NAME}_PORT /dev/tty.usbmodem1d11)

Serial Terminal
~~~~~~~~~~~~~~~

On Mac the easiest way to get a Serial Terminal is to use the ``screen`` terminal emulator. To start a ``screen`` serial session::

    screen /dev/tty.usbmodemXXX

Where ``/dev/tty.usbmodemXXX`` is the terminal device. To exit press ``C-a C-\``.

CMake configuration example::

    set(${FIRMWARE_NAME}_SERIAL screen @INPUT_PORT@)











Windows Environment Setup
-------------------------

On Windows the *Arduino SDK* is self contained and has everything needed for building. To setup the environment do the following:

1. Place the `Arduino SDK`_ either
   
   * into  **Program Files**, or
   * onto the **System Path**
    
   NOTE: Don't change the default *Arduino SDK* directory name, otherwise auto detection will no work properly!

2. Add to the **System Path**: ``${ARDUINO_SDK_PATH}/hardware/tools/avr/utils/bin``
3. Install `CMake 2.8`_
   
   NOTE: Make sure you check the option to add CMake to the **System Path**.


CMake Generators
~~~~~~~~~~~~~~~~

Once installed, you can start using CMake the usual way, just make sure to chose either a **MSYS Makefiles** or **Unix Makefiles** type generator::

    MSYS Makefiles              = Generates MSYS makefiles.
    Unix Makefiles              = Generates standard UNIX makefiles.
    CodeBlocks - Unix Makefiles = Generates CodeBlocks project files.
    Eclipse CDT4 - Unix Makefiles
                                = Generates Eclipse CDT 4.0 project files.

If you want to use a **MinGW Makefiles** type generator, you must generate the build system the following way:

1. Remove ``${ARDUINO_SDK_PATH}/hardware/tools/avr/utils/bin`` from the **System Path**
2. Generate the build system using CMake with the following option set (either through the GUI or from the command line)::

    CMAKE_MAKE_PROGRAM=${ARDIUNO_SDK_PATH}/hardware/tools/avr/utils/bin/make.exe

3. Then build the normal way

The reason for doing this is the MinGW generator cannot have the ``sh.exe`` binary on the **System Path** during generation, otherwise you get an error.

Serial Naming
~~~~~~~~~~~~~

When specifying the serial port name on Windows, use the following names::

    com1 com2 ... comN

CMake configuration example::

    set(${FIRMWARE_NAME}_PORT com3)

Serial Terminal
~~~~~~~~~~~~~~~

Putty is a great multi-protocol terminal, which supports SSH, Telnet, Serial, and many more... The latest development snapshot supports command line options for launching a serial terminal, for example::

    putty -serial COM3 -sercfg 9600,8,n,1,X

CMake configuration example (assuming putty is on the **System Path**)::

    set(${FIRMWARE_NAME}_SERIAL putty -serial @INPUT_PORT@)

Putty - http://tartarus.org/~simon/putty-snapshots/x86/putty-installer.exe










Eclipse Environment
-------------------

Eclipse is a great IDE which has a lot of functionality and is much more powerful than the *Arduino IDE*. In order to use Eclipse you will need the following:

1. Eclipse
2. Eclipse CDT extension (for C/C++ development)

On most Linux distribution you can install Eclipse + CDT using your package manager, otherwise you can download the `Eclipse IDE for C/C++ Developers`_ bundle.

Once you have Eclipse, here is how to generate a project using CMake:

1. Create a build directory that is next to your source directory, like this::
   
       build_directory/
       source_directory/

2. Run CMake with the `Eclipse CDT4 - Unix Makefiles` generator, inside the build directory::

        cd build_directory/
        cmake -G"Eclipse CDT4 - Unix Makefiles" ../source_directory

3. Open Eclipse and import the project from the build directory.

   1. **File > Import**
   2. Select `Existing Project into Workspace`, and click **Next**
   3. Select *Browse*, and select the build directoy.
   4. Select the project in the **Projects:** list
   5. Click **Finish**



.. _Eclipse IDE for C/C++ Developers: http://www.eclipse.org/downloads/packages/eclipse-ide-cc-developers/heliossr2











Troubleshooting
---------------

The following section will outline some solutions to common problems that you may encounter.

undefined reference to `__cxa_pure_virtual'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When linking you'r firmware image you may encounter this error on some systems. An easy fix is to add the following to your firmware source code::

    extern "C" void __cxa_pure_virtual(void);
    void __cxa_pure_virtual(void) { while(1); } 


The contents of the ``__cxa_pure_virtual`` function can be any error handling code; this function will be called whenever a pure virtual function is called. 

* `What is the purpose of `cxa_pure_virtual``_

.. _What is the purpose of `cxa_pure_virtual`: http://stackoverflow.com/questions/920500/what-is-the-purpose-of-cxa-pure-virtual

Arduino Mega 2560 image does not work
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are working on Linux, and have ``avr-gcc`` >= 4.5 you might have a unpatched version gcc which has the C++ constructor bug. This bug affects the **Atmega2560** when using classes which causes the Arduino firmware to crash.

If you encounter this problem either downgrade ``avr-gcc`` to **4.3** or rebuild gcc with the following patch::

    --- gcc-4.5.1.orig/gcc/config/avr/libgcc.S  2009-05-23 17:16:07 +1000
    +++ gcc-4.5.1/gcc/config/avr/libgcc.S   2010-08-12 09:38:05 +1000
    @@ -802,7 +802,9 @@
        mov_h   r31, r29
        mov_l   r30, r28
        out     __RAMPZ__, r20
    +   push    r20
        XCALL   __tablejump_elpm__
    +   pop r20
     .L__do_global_ctors_start:
        cpi r28, lo8(__ctors_start)
        cpc r29, r17
    @@ -843,7 +845,9 @@
        mov_h   r31, r29
        mov_l   r30, r28
        out     __RAMPZ__, r20
    +   push    r20
        XCALL   __tablejump_elpm__
    +   pop r20
     .L__do_global_dtors_start:
        cpi r28, lo8(__dtors_end)
        cpc r29, r17

* `AVR GCC Bug 45263 Report`_
* `The global constructor bug in avr-gcc`_

.. _AVR GCC Bug 45263 Report: http://gcc.gnu.org/bugzilla/show_bug.cgi?id=45263
.. _The global constructor bug in avr-gcc: http://andybrown.me.uk/ws/2010/10/24/the-major-global-constructor-bug-in-avr-gcc/



Library not detected automatically
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a Arduino library does not get detected automatically, it usually means CMake cannot find it (obvious).

One common reason why the library is not detected, is because the directory name of the library does not match the header.
If I'm including a library header like so::

    #include "my_library.h"

Based on this include, **Arduino CMake** is expecting to find a library that has a directory name **my_libray**.
If the directory name does not match the header, it won't be consider a Arduino Library (see `Arduino Libraries`_).


When a library being used is located in a non-standard location (not in the **Arduino SDK** or next to the firmware), then that directory must be registered.
To register a non-standard directory containing Arduino libraries, use the following::

    link_directories(path_to_directory_containing_libraries)

Remember to **use this command before defining any firmware**, depending on libraries located in that directory.




Resources
---------

Here are some resources you might find useful in getting started.

1. CMake:

   * `Offical CMake Tutorial`_
   * `CMake Tutorial`_
   * `CMake Reference`_

.. _Offical CMake Tutorial: http://www.cmake.org/cmake/help/cmake_tutorial.html
.. _CMake Tutorial: http://mathnathan.com/2010/07/11/getting-started-with-cmake/
.. _CMake Reference: http://www.cmake.org/cmake/help/cmake-2-8-docs.html

2. Arduino:
   
   * `Getting Started`_ - Introduction to Arduino
   * `Playground`_ - User contributed documentation and help
   * `Arduino Forums`_ - Official forums
   * `Arduino Reference`_ - Official reference manual

.. _Getting Started: http://www.arduino.cc/en/Guide/HomePage
.. _Playground: http://www.arduino.cc/playground/
.. _Arduino Reference: http://www.arduino.cc/en/Reference/HomePage
.. _Arduino Forums: http://www.arduino.cc/forum/








.. _CMake 2.8: http://www.cmake.org/cmake/resources/software.html
.. _CMake: http://www.cmake.org/cmake/resources/software.html
.. _CMake Installer: http://www.cmake.org/cmake/resources/software.html
.. _Arduino SDK: http://www.arduino.cc/en/Main/Software

