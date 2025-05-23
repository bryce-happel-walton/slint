# CMake Reference
<!-- Copyright © SixtyFPS GmbH <info@slint.dev> ; SPDX-License-Identifier: MIT -->

## `slint_target_sources`

```
slint_target_sources(<target> <files>.... [NAMESPACE namespace] [LIBRARY_PATHS name1=lib1 name2=lib2 ...] [COMPILATION_UNITS num])
```

Use this function to tell CMake about the .slint files of your application, similar to the builtin cmake [target_sources](https://cmake.org/cmake/help/latest/command/target_sources.html) function.
The function takes care of running the slint-compiler to convert `.slint` files to `.h` files in the build directory,
and extend the include directories of your target so that the generated file is found when including it in your application.

The optional `NAMESPACE` argument will put the generated components in the given C++ namespace.

Use the `LIBRARY_PATHS` argument to specify the name and paths to {{ '[component libraries]({})'.format(slint_href_ComponentLibraries) }},
separated by an equals sign (`=`).

Given a file called `the_window.slint`, the following example will create a file called `the_window.h` that can
be included from your .cpp file. Assuming the `the_window.slint` contains a component `TheWindow`, the output
C++ class will be put in the namespace `ui`, resulting to `ui::TheWindow`. Any import from `@mycomponentlib/` will
be redirected to the specified path.

```cmake
add_executable(my_application main.cpp)
target_link_libraries(my_application PRIVATE Slint::Slint)
slint_target_sources(my_application the_window.slint
    NAMESPACE ui
    LIBRARY_PATHS mycomponentlib=/path/to/customcomponents
)
```

By default, a `.slint` file is compiled to a `.h` file for inclusion in your application's business logic code, and a `.cpp` file with code generated by
the slint-compiler. If you want to speed up compilation of the generated `.cpp` file, then you can pass the `COMPILATION_UNITS` argument with a value greater
than 1 to create multiple `.cpp` files. These can be compiled in parallel, which might speed up overall build times. However, splitting the generated code
across multiple `.cpp` files decreases the compiler's visibility and thus ability to perform optimizations. You can also pass `COMPILATION_UNITS 0` to generate
only one single `.h` file.

## Resource Embedding

By default, images from {{ '[`@image-url()`]({})'.format(slint_href_ImageType) }} or fonts that your Slint files reference are loaded from disk at run-time. This minimises build times, but requires that the directory structure with the files remains stable. If you want to build a program that runs anywhere, then you can configure the Slint compiler to embed such sources into the binary.

Set the `SLINT_EMBED_RESOURCES` target property on your CMake target to one of the following values:

* `embed-files`: The raw files are embedded in the application binary.
* `embed-for-software-renderer`: The files will be loaded by the Slint compiler, optimized for use with the software renderer and embedded in the application binary.
* `embed-for-software-renderer-with-sdf`: Same as `embed-for-software-renderer`, but use [Signed Distance Fields (SDF)](https://en.wikipedia.org/wiki/Signed_distance_function) to render fonts.
  This produces smaller binaries, but may result in slightly inferior visual output and slower rendering.
  (Requires the `SLINT_FEATURE_SDF_FONTS` feature to be enabled.)
* `as-absolute-path`: The paths of files are made absolute and will be used at run-time to load the resources from the file system. This is the default.

This target property is initialised from the global `DEFAULT_SLINT_EMBED_RESOURCES` cache variable. Set it to configure the default for all CMake targets.

```cmake
# Example: when building my_application, specify that the compiler should embed the resources in the binary
set_property(TARGET my_application PROPERTY SLINT_EMBED_RESOURCES embed-files)
```

## Scale Factor for Microcontrollers

When targeting a Microcontroller, there exists no windowing system that provides a device pixel ratio to
map logical lengths in Slint (`px`) to physical pixels (`phx`). If desired, you can provide this ratio at
compile time by setting the `SLINT_SCALE_FACTOR` target property on your CMake target.

```cmake
# Example: when building my_application, specify that the scale factor shall be 2
set_property(TARGET my_application PROPERTY SLINT_SCALE_FACTOR 2.0)
```

A scale factor specified this way will also be used to pre-scale images and glyphs when used in combination
with [Resource Embedding](#resource-embedding).

## Bundle Translations

Translations can either be done using `gettext` at runtime, or by bundling all the translated strings
directly into the binary, by embedding them in the generated C++ code.
If you want to bundle translations, you need to set the `SLINT_BUNDLE_TRANSLATIONS` target property
to point to a directory containing translations. The translations must be in the gettext `.po` format.

In the following example, the translation files will be bundled from `lang/<lang>/LC_MESSAGES/my_application.po`

```cmake
set_property(TARGET my_application PROPERTY SLINT_BUNDLE_TRANSLATIONS "${CMAKE_CURRENT_SOURCE_DIR}/lang")
```

## Translation Domain

By default, the domain used for translations is the name of the CMake target the `.slint` files are targeted with.
Use the `SLINT_TRANSLATION_DOMAIN` target property to override this and use the specified value as domain, instead.
This is useful in build environments where the target name is given and not suitable, such as esp-idf.
