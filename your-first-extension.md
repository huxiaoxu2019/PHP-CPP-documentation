# 你的第一个扩展
当你创建自己的PHP-CPP扩展时，同样你不得不编译和部署它。一个正常的PHP脚本仅仅需要拷贝到Web服务上即可部署，但是部署一个扩展需要花费一些精力：你需要一个Makefile文件，一个用于扩展的php.ini文件，当然还有一个实现扩展业务的*.cpp文件。

为了让你清楚这些步骤，我们已经创建了一个几乎为空的扩展，但包含了所有这些必须的文件。它包含了一个Makefile文件，配置文件和一个已经实现get_module（）调用的主函数文件main.cpp。这些会帮助你快速的开始扩展开发。

- [EmptyExtension.zip](http://www.php-cpp.com/EmptyExtension.zip)
- [EmptyExtension.tar.gz](http://www.php-cpp.com/EmptyExtension.tar.gz)

## Makefile
上述的EmptyExtension文件中包含了一个详尽描述编译器的Makefile文件。为了满足你的扩展，你需要（简单地）修改下Makefile文件。其中最重要的修改地方就是扩展名称，也许还有INI_DIR配置。

```
#
#   Makefile template
#
#   This is an example Makefile that can be used by anyone who is building
#   his or her own PHP extensions using the PHP-CPP library. 
#
#   In the top part of this file we have included variables that can be
#   altered to fit your configuration, near the bottom the instructions and
#   dependencies for the compiler are defined. The deeper you get into this
#   file, the less likely it is that you will have to change anything in it.
#

#
#   Name of your extension
#
#   This is the name of your extension. Based on this extension name, the
#   name of the library file (name.so) and the name of the config file (name.ini)
#   are automatically generated
#

NAME                =   yourextension

#
#   Php.ini directories
#
#   In the past, PHP used a single php.ini configuration file. Today, most
#   PHP installations use a conf.d directory that holds a set of config files,
#   one for each extension. Use this variable to specify this directory.
#
#   In Ubuntu 14.04 Apache 2.4 is used, which uses the mods-available directory
#   instead of a conf.d directory. In 16.04 the directory changed yet again.
#   This has to be checked.
#

UBUNTU_MAJOR  := $(shell /usr/bin/lsb_release -r -s | cut -f1 -d.)
OVER_SIXTEEN  := $(shell echo "${UBUNTU_MAJOR} >= 16" | bc)
OVER_FOURTEEN := $(shell echo "${UBUNTU_MAJOR} >= 14" | bc)

ifeq (${OVER_SIXTEEN}, 1)
    INI_DIR     =   /etc/php/7.0/mods-available/
else ifeq (${OVER_FOURTEEN}, 1)
    INI_DIR     =   /etc/php5/mods-available/
else
    INI_DIR     =   /etc/php5/conf.d/
endif

#
#   The extension dirs
#
#   This is normally a directory like /usr/lib/php5/20121221 (based on the 
#   PHP version that you use. We make use of the command line 'php-config' 
#   instruction to find out what the extension directory is, you can override
#   this with a different fixed directory
#

EXTENSION_DIR       =   $(shell php-config --extension-dir)

#
#   The name of the extension and the name of the .ini file
#
#   These two variables are based on the name of the extension. We simply add
#   a certain extension to them (.so or .ini)
#

EXTENSION           =   ${NAME}.so
INI                 =   ${NAME}.ini

#
#   Compiler
#
#   By default, the GNU C++ compiler is used. If you want to use a different
#   compiler, you can change that here. You can change this for both the 
#   compiler (the program that turns the c++ files into object files) and for
#   the linker (the program that links all object files into the single .so
#   library file. By default, g++ (the GNU C++ compiler) is used for both.
#

COMPILER            =   g++
LINKER              =   g++

#
#   Compiler and linker flags
#
#   This variable holds the flags that are passed to the compiler. By default, 
#   we include the -O2 flag. This flag tells the compiler to optimize the code, 
#   but it makes debugging more difficult. So if you're debugging your application, 
#   you probably want to remove this -O2 flag. At the same time, you can then 
#   add the -g flag to instruct the compiler to include debug information in
#   the library (but this will make the final libphpcpp.so file much bigger, so
#   you want to leave that flag out on production servers).
#
#   If your extension depends on other libraries (and it does at least depend on
#   one: the PHP-CPP library), you should update the LINKER_DEPENDENCIES variable
#   with a list of all flags that should be passed to the linker.
#

COMPILER_FLAGS      =   -Wall -c -O2 -std=c++11 -fpic -o
LINKER_FLAGS        =   -shared
LINKER_DEPENDENCIES =   -lphpcpp

#
#   Command to remove files, copy files and create directories.
#
#   I've never encountered a *nix environment in which these commands do not work. 
#   So you can probably leave this as it is
#

RM                  =   rm -f
CP                  =   cp -f
MKDIR               =   mkdir -p

#
#   All source files are simply all *.cpp files found in the current directory
#
#   A built-in Makefile macro is used to scan the current directory and find 
#   all source files. The object files are all compiled versions of the source
#   file, with the .cpp extension being replaced by .o.
#

SOURCES             =   $(wildcard *.cpp)
OBJECTS             =   $(SOURCES:%.cpp=%.o)

#
#   From here the build instructions start
#

all:                    ${OBJECTS} ${EXTENSION}

${EXTENSION}:           ${OBJECTS}
                        ${LINKER} ${LINKER_FLAGS} -o $@ ${OBJECTS} ${LINKER_DEPENDENCIES}

${OBJECTS}:
                        ${COMPILER} ${COMPILER_FLAGS} $@ ${@:%.o=%.cpp}

install:        
                        ${CP} ${EXTENSION} ${EXTENSION_DIR}
                        ${CP} ${INI} ${INI_DIR}

clean:
                        ${RM} ${EXTENSION} ${OBJECTS}
```

## Yourextension.ini
在你的扩展中，除了Makefile文件，还应该有一个yourextension.ini文件。这个文件相对于Makefile文件更为简单。同样，你需要把扩展名称改成你想要的（yourextension.so例外）。

```
extension=yourextension.so
```

## Main.cpp
在EmptyExtension包中最后一个有用的文件是Main.cpp，它实现了扩展的业务。由于扩展项目是空的，所以需要对Main.cpp做大量的改动：你需要向Main.cpp中加入类和函数。

```
#include <phpcpp.h>

/**
 *  tell the compiler that the get_module is a pure C function
 */
extern "C" {

    /**
     *  Function that is called by PHP right after the PHP process
     *  has started, and that returns an address of an internal PHP
     *  strucure with all the details and features of your extension
     *
     *  @return void*   a pointer to an address that is understood by PHP
     */
    PHPCPP_EXPORT void *get_module() 
    {
        // static(!) Php::Extension object that should stay in memory
        // for the entire duration of the process (that's why it's static)
        static Php::Extension extension("yourextension", "1.0");

        // @todo    add your own functions, classes, namespaces to the extension

        // return the extension
        return extension;
    }
}
```
