### Steps

- Step 1

  - tutorial.cxx:

        // A simple program that computes the square root of a number
        #include <cmath>
        #include <iostream>
        #include <string>

        #include "TutorialConfig.h"

        int main(int argc, char\* argv[])
        {
        if (argc < 2) {
        // report version
        std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
        << Tutorial_VERSION_MINOR << std::endl;
        std::cout << "Usage: " << argv[0] << " number" << std::endl;
        return 1;
        }

        // convert input to double
        const double inputValue = std::stod(argv[1]);

        // calculate square root
        const double outputValue = sqrt(inputValue);
        std::cout << "The square root of " << inputValue << " is " << outputValue
        << std::endl;
        return 0;
        }

  - CMakeLists.txt:

        cmake_minimum_required(VERSION 3.10)

        # set the project name
        project(Tutorial VERSION 1.0)

        # specify the C++ standard
        set(CMAKE_CXX_STANDARD 11)
        set(CMAKE_CXX_STANDARD_REQUIRED True)

        configure_file(TutorialConfig.h.in TutorialConfig.h)

        # add the executable
        add_executable(Tutorial tutorial.cxx)

        target_include_directories(Tutorial PUBLIC
                                "${PROJECT_BINARY_DIR}"
                                )

  - Tutorial:

    ![tutorial](step1.PNG)

- Step 2

  - tutorial.cxx:

        // A simple program that computes the square root of a number
        #include <cmath>
        #include <iostream>
        #include <string>

        #include "TutorialConfig.h"

        #ifdef USE_MYMATH
        #  include "MathFunctions.h"
        #endif

        int main(int argc, char* argv[])
        {
        if (argc < 2) {
            // report version
            std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
                    << Tutorial_VERSION_MINOR << std::endl;
            std::cout << "Usage: " << argv[0] << " number" << std::endl;
            return 1;
        }

        // convert input to double
        const double inputValue = std::stod(argv[1]);

        #ifdef USE_MYMATH
        const double outputValue = mysqrt(inputValue);
        #else
        const double outputValue = sqrt(inputValue);
        #endif
        std::cout << "The square root of " << inputValue << " is " << outputValue
                    << std::endl;
        return 0;
        }

  - CMakeLists.txt:

        cmake_minimum_required(VERSION 3.10)

        # set the project name and version
        project(Tutorial VERSION 1.0)

        # specify the C++ standard
        set(CMAKE_CXX_STANDARD 11)
        set(CMAKE_CXX_STANDARD_REQUIRED True)

        option(USE_MYMATH "Use tutorial provided math implementation" ON)

        # configure a header file to pass some of the CMake settings
        # to the source code
        configure_file(TutorialConfig.h.in TutorialConfig.h)

        if(USE_MYMATH)
        add_subdirectory(MathFunctions)
        list(APPEND EXTRA_LIBS MathFunctions)
        list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
        endif()

        # add the executable
        add_executable(Tutorial tutorial.cxx)

        target_link_libraries(Tutorial PUBLIC MathFunctions)

        # add the binary tree to the search path for include files
        # so that we will find TutorialConfig.h
        target_include_directories(Tutorial PUBLIC
                                "${PROJECT_BINARY_DIR}"
                                "${PROJECT_SOURCE_DIR}/MathFunctions"
                                )

  - Tutorial:

    ![tutorial](step2.PNG)

- Step 3

  - MathFunctions/CMakeLists.txt:

        add_library(MathFunctions mysqrt.cxx)

        target_include_directories(MathFunctions
                INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                )

  - CMakeLists.txt:

        cmake_minimum_required(VERSION 3.10)

        # set the project name and version
        project(Tutorial VERSION 1.0)

        # specify the C++ standard
        set(CMAKE_CXX_STANDARD 11)
        set(CMAKE_CXX_STANDARD_REQUIRED True)

        # should we use our own math functions
        option(USE_MYMATH "Use tutorial provided math implementation" ON)

        # configure a header file to pass some of the CMake settings
        # to the source code
        configure_file(TutorialConfig.h.in TutorialConfig.h)

        # add the MathFunctions library
        if(USE_MYMATH)
        add_subdirectory(MathFunctions)
        list(APPEND EXTRA_LIBS MathFunctions)
        endif()

        # add the executable
        add_executable(Tutorial tutorial.cxx)

        target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

        # add the binary tree to the search path for include files
        # so that we will find TutorialConfig.h
        target_include_directories(Tutorial PUBLIC
                                "${PROJECT_BINARY_DIR}"
                                )

  - Tutorial:

    ![tutorial](step3.PNG)

- Step 4

  - MathFunctions/CMakeLists.txt:

        add_library(MathFunctions mysqrt.cxx)

        # state that anybody linking to us needs to include the current source dir
        # to find MathFunctions.h, while we don't.
        target_include_directories(MathFunctions
                INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                )

        install(TARGETS MathFunctions DESTINATION lib)
        install(FILES MathFunctions.h DESTINATION include)

  - CMakeLists.txt:

        cmake_minimum_required(VERSION 3.10)

        # set the project name and version
        project(Tutorial VERSION 1.0)

        # specify the C++ standard
        set(CMAKE_CXX_STANDARD 11)
        set(CMAKE_CXX_STANDARD_REQUIRED True)

        # should we use our own math functions
        option(USE_MYMATH "Use tutorial provided math implementation" ON)

        # configure a header file to pass some of the CMake settings
        # to the source code
        configure_file(TutorialConfig.h.in TutorialConfig.h)

        # add the MathFunctions library
        if(USE_MYMATH)
        add_subdirectory(MathFunctions)
        list(APPEND EXTRA_LIBS MathFunctions)
        endif()

        # add the executable
        add_executable(Tutorial tutorial.cxx)

        target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

        # add the binary tree to the search path for include files
        # so that we will find TutorialConfig.h
        target_include_directories(Tutorial PUBLIC
                                "${PROJECT_BINARY_DIR}"
                                )

        install(TARGETS Tutorial DESTINATION bin)
        install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include
        )

        enable_testing()

        # does the application run
        add_test(NAME Runs COMMAND Tutorial 25)

        # does the usage message work?
        add_test(NAME Usage COMMAND Tutorial)
        set_tests_properties(Usage
        PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
        )

        # define a function to simplify adding tests
        function(do_test target arg result)
        add_test(NAME Comp${arg} COMMAND ${target} ${arg})
        set_tests_properties(Comp${arg}
            PROPERTIES PASS_REGULAR_EXPRESSION ${result}
            )
        endfunction(do_test)

        # do a bunch of result based tests
        do_test(Tutorial 4 "4 is 2")
        do_test(Tutorial 9 "9 is 3")
        do_test(Tutorial 5 "5 is 2.236")
        do_test(Tutorial 7 "7 is 2.645")
        do_test(Tutorial 25 "25 is 5")
        do_test(Tutorial -25 "-25 is [-nan|nan|0]")
        do_test(Tutorial 0.0001 "0.0001 is 0.01")

  - ctest -W: ![tutorial](step4.PNG)

- Step 5

  - MathFunctions/CMakeLists.txt:

        add_library(MathFunctions mysqrt.cxx)

        # state that anybody linking to us needs to include the current source dir
        # to find MathFunctions.h, while we don't.
        target_include_directories(MathFunctions
                INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                )

        include(CheckSymbolExists)
        check_symbol_exists(log "math.h" HAVE_LOG)
        check_symbol_exists(exp "math.h" HAVE_EXP)
        if(NOT (HAVE_LOG AND HAVE_EXP))
        unset(HAVE_LOG CACHE)
        unset(HAVE_EXP CACHE)
        set(CMAKE_REQUIRED_LIBRARIES "m")
        check_symbol_exists(log "math.h" HAVE_LOG)
        check_symbol_exists(exp "math.h" HAVE_EXP)
        if(HAVE_LOG AND HAVE_EXP)
            target_link_libraries(MathFunctions PRIVATE m)
        endif()
        endif()

        if(HAVE_LOG AND HAVE_EXP)
        target_compile_definitions(MathFunctions
                                    PRIVATE "HAVE_LOG" "HAVE_EXP")
        endif()

        # install rules
        install(TARGETS MathFunctions DESTINATION lib)
        install(FILES MathFunctions.h DESTINATION include)

  - CMakeLists.txt:

        cmake_minimum_required(VERSION 3.10)

        # set the project name and version
        project(Tutorial VERSION 1.0)

        # specify the C++ standard
        set(CMAKE_CXX_STANDARD 11)
        set(CMAKE_CXX_STANDARD_REQUIRED True)

        # should we use our own math functions
        option(USE_MYMATH "Use tutorial provided math implementation" ON)

        # configure a header file to pass some of the CMake settings
        # to the source code
        configure_file(TutorialConfig.h.in TutorialConfig.h)

        # add the MathFunctions library
        if(USE_MYMATH)
        add_subdirectory(MathFunctions)
        list(APPEND EXTRA_LIBS MathFunctions)
        endif()

        # add the executable
        add_executable(Tutorial tutorial.cxx)
        target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

        # add the binary tree to the search path for include files
        # so that we will find TutorialConfig.h
        target_include_directories(Tutorial PUBLIC
                                "${PROJECT_BINARY_DIR}"
                                )

        # add the install targets
        install(TARGETS Tutorial DESTINATION bin)
        install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include
        )

        # enable testing
        enable_testing()

        # does the application run
        add_test(NAME Runs COMMAND Tutorial 25)

        # does the usage message work?
        add_test(NAME Usage COMMAND Tutorial)
        set_tests_properties(Usage
        PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
        )

        # define a function to simplify adding tests
        function(do_test target arg result)
        add_test(NAME Comp${arg} COMMAND ${target} ${arg})
        set_tests_properties(Comp${arg}
            PROPERTIES PASS_REGULAR_EXPRESSION ${result}
            )
        endfunction(do_test)

        # do a bunch of result based tests
        do_test(Tutorial 4 "4 is 2")
        do_test(Tutorial 9 "9 is 3")
        do_test(Tutorial 5 "5 is 2.236")
        do_test(Tutorial 7 "7 is 2.645")
        do_test(Tutorial 25 "25 is 5")
        do_test(Tutorial -25 "-25 is [-nan|nan|0]")
        do_test(Tutorial 0.0001 "0.0001 is 0.01")

  - Tutorial:

    ![tutorial](step5.PNG)

- When you are finished, get a copy of the code at [https://github.com/rcos/CSCI-4470-OpenSource/tree/master/Modules/05.BuildSystems/Lab-BuildSystemsExample](https://github.com/rcos/CSCI-4470-OpenSource/tree/master/Modules/05.BuildSystems/Lab-BuildSystemsExample). If you have the code checked out, do a fresh pull. I removed some code that had been mistakenly committed.

  - In your lab report, paste in your Makefile, your CMakeLists.txt file, the Makefile created by cmake, the relative size of your two executables, and the results of running your program.

    - Makefile:

            CC=gcc
            PROJDIR := $(realpath $(CURDIR)/..)

            all: program
            program: program.o dynamic_block.so static_block.a
            program.o: program.c
            dynamic_block.so: block.o
            static_block.a: block.o
            block.o: block.c
            cc -fPIC -c block.c -o block.o -Wl,-rpath='$(PROJDIR)/source'

    - CMakeLists:

            cmake_minimum_required(VERSION 3.10)

            # set the project name
            project(Program1)

            add_library(static_block STATIC source/block.c )

            # add the executable
            add_executable(Program1 program.c)

            target_link_libraries(Program1 static_block)

            # set the project name
            project(Program2)

            add_library(dynamic_block SHARED source/block.c )

            # add the executable
            add_executable(Program2 program.c)

            target_link_libraries(Program2 dynamic_block)

    - Makefile of CMakeLists:

                  # CMAKE generated file: DO NOT EDIT!
                  # Generated by "Unix Makefiles" Generator, CMake Version 3.10

                  # Default target executed when no arguments are given to make.
                  default_target: all

                  .PHONY : default_target

                  # Allow only one "make -f Makefile2" at a time, but pass parallelism.
                  .NOTPARALLEL:


                  #=============================================================================
                  # Special targets provided by cmake.

                  # Disable implicit rules so canonical targets will work.
                  .SUFFIXES:


                  # Remove some rules from gmake that .SUFFIXES does not remove.
                  SUFFIXES =

                  .SUFFIXES: .hpux_make_needs_suffix_list


                  # Suppress display of executed commands.
                  $(VERBOSE).SILENT:


                  # A target that is always out of date.
                  cmake_force:

                  .PHONY : cmake_force

                  #=============================================================================
                  # Set environment variables for the build.

                  # The shell in which to execute make rules.
                  SHELL = /bin/sh

                  # The CMake executable.
                  CMAKE_COMMAND = /usr/bin/cmake

                  # The command to remove a file.
                  RM = /usr/bin/cmake -E remove -f

                  # Escaping for special characters.
                  EQUALS = =

                  # The top-level source directory on which CMake was run.
                  CMAKE_SOURCE_DIR = "/mnt/c/users/Jacob Zamani/documents/2020-2021/Spring2021/OpenSource/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample"

                  # The top-level build directory on which CMake was run.
                  CMAKE_BINARY_DIR = "/mnt/c/users/Jacob Zamani/documents/2020-2021/Spring2021/OpenSource/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample/build"

                  #=============================================================================
                  # Targets provided globally by CMake.

                  # Special rule for the target edit_cache
                  edit_cache:
                        @$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --cyan "No interactive CMake dialog available..."
                        /usr/bin/cmake -E echo No\ interactive\ CMake\ dialog\ available.
                  .PHONY : edit_cache

                  # Special rule for the target edit_cache
                  edit_cache/fast: edit_cache

                  .PHONY : edit_cache/fast

                  # Special rule for the target rebuild_cache
                  rebuild_cache:
                        @$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --cyan "Running CMake to regenerate build system..."
                        /usr/bin/cmake -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR)
                  .PHONY : rebuild_cache

                  # Special rule for the target rebuild_cache
                  rebuild_cache/fast: rebuild_cache

                  .PHONY : rebuild_cache/fast

                  # The main all target
                  all: cmake_check_build_system
                        $(CMAKE_COMMAND) -E cmake_progress_start "/mnt/c/users/Jacob Zamani/documents/2020-2021/Spring2021/OpenSource/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample/build/CMakeFiles" "/mnt/c/users/Jacob Zamani/documents/2020-2021/Spring2021/OpenSource/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample/build/CMakeFiles/progress.marks"
                        $(MAKE) -f CMakeFiles/Makefile2 all
                        $(CMAKE_COMMAND) -E cmake_progress_start "/mnt/c/users/Jacob Zamani/documents/2020-2021/Spring2021/OpenSource/CSCI-4470-OpenSource/Modules/05.BuildSystems/Lab-BuildSystemsExample/build/CMakeFiles" 0
                  .PHONY : all

                  # The main clean target
                  clean:
                        $(MAKE) -f CMakeFiles/Makefile2 clean
                  .PHONY : clean

                  # The main clean target
                  clean/fast: clean

                  .PHONY : clean/fast

                  # Prepare targets for installation.
                  preinstall: all
                        $(MAKE) -f CMakeFiles/Makefile2 preinstall
                  .PHONY : preinstall

                  # Prepare targets for installation.
                  preinstall/fast:
                        $(MAKE) -f CMakeFiles/Makefile2 preinstall
                  .PHONY : preinstall/fast

                  # clear depends
                  depend:
                        $(CMAKE_COMMAND) -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 1
                  .PHONY : depend

                  #=============================================================================
                  # Target rules for targets named Program2

                  # Build rule for target.
                  Program2: cmake_check_build_system
                        $(MAKE) -f CMakeFiles/Makefile2 Program2
                  .PHONY : Program2

                  # fast build rule for target.
                  Program2/fast:
                        $(MAKE) -f CMakeFiles/Program2.dir/build.make CMakeFiles/Program2.dir/build
                  .PHONY : Program2/fast

                  #=============================================================================
                  # Target rules for targets named dynamic_block

                  # Build rule for target.
                  dynamic_block: cmake_check_build_system
                        $(MAKE) -f CMakeFiles/Makefile2 dynamic_block
                  .PHONY : dynamic_block

                  # fast build rule for target.
                  dynamic_block/fast:
                        $(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/build
                  .PHONY : dynamic_block/fast

                  #=============================================================================
                  # Target rules for targets named static_block

                  # Build rule for target.
                  static_block: cmake_check_build_system
                        $(MAKE) -f CMakeFiles/Makefile2 static_block
                  .PHONY : static_block

                  # fast build rule for target.
                  static_block/fast:
                        $(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/build
                  .PHONY : static_block/fast

                  #=============================================================================
                  # Target rules for targets named Program1

                  # Build rule for target.
                  Program1: cmake_check_build_system
                        $(MAKE) -f CMakeFiles/Makefile2 Program1
                  .PHONY : Program1

                  # fast build rule for target.
                  Program1/fast:
                        $(MAKE) -f CMakeFiles/Program1.dir/build.make CMakeFiles/Program1.dir/build
                  .PHONY : Program1/fast

                  program.o: program.c.o

                  .PHONY : program.o

                  # target to build an object file
                  program.c.o:
                        $(MAKE) -f CMakeFiles/Program2.dir/build.make CMakeFiles/Program2.dir/program.c.o
                        $(MAKE) -f CMakeFiles/Program1.dir/build.make CMakeFiles/Program1.dir/program.c.o
                  .PHONY : program.c.o

                  program.i: program.c.i

                  .PHONY : program.i

                  # target to preprocess a source file
                  program.c.i:
                        $(MAKE) -f CMakeFiles/Program2.dir/build.make CMakeFiles/Program2.dir/program.c.i
                        $(MAKE) -f CMakeFiles/Program1.dir/build.make CMakeFiles/Program1.dir/program.c.i
                  .PHONY : program.c.i

                  program.s: program.c.s

                  .PHONY : program.s

                  # target to generate assembly for a file
                  program.c.s:
                        $(MAKE) -f CMakeFiles/Program2.dir/build.make CMakeFiles/Program2.dir/program.c.s
                        $(MAKE) -f CMakeFiles/Program1.dir/build.make CMakeFiles/Program1.dir/program.c.s
                  .PHONY : program.c.s

                  source/block.o: source/block.c.o

                  .PHONY : source/block.o

                  # target to build an object file
                  source/block.c.o:
                        $(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/source/block.c.o
                        $(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/source/block.c.o
                  .PHONY : source/block.c.o

                  source/block.i: source/block.c.i

                  .PHONY : source/block.i

                  # target to preprocess a source file
                  source/block.c.i:
                        $(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/source/block.c.i
                        $(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/source/block.c.i
                  .PHONY : source/block.c.i

                  source/block.s: source/block.c.s

                  .PHONY : source/block.s

                  # target to generate assembly for a file
                  source/block.c.s:
                        $(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/source/block.c.s
                        $(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/source/block.c.s
                  .PHONY : source/block.c.s

                  # Help Target
                  help:
                        @echo "The following are some of the valid targets for this Makefile:"
                        @echo "... all (the default if no target is provided)"
                        @echo "... clean"
                        @echo "... depend"
                        @echo "... edit_cache"
                        @echo "... Program2"
                        @echo "... dynamic_block"
                        @echo "... static_block"
                        @echo "... rebuild_cache"
                        @echo "... Program1"
                        @echo "... program.o"
                        @echo "... program.i"
                        @echo "... program.s"
                        @echo "... source/block.o"
                        @echo "... source/block.i"
                        @echo "... source/block.s"
                  .PHONY : help



                  #=============================================================================
                  # Special targets to cleanup operation of make.

                  # Special rule to run CMake to check the build system integrity.
                  # No rule that depends on this can have commands that come from listfiles
                  # because they might be regenerated.
                  cmake_check_build_system:
                        $(CMAKE_COMMAND) -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 0
                  .PHONY : cmake_check_build_system

    - Size of dynamic library: 8269 bytes
    - Size of static library: 8464 bytes
    - Run Results:

    ![results](runresults.PNG)
