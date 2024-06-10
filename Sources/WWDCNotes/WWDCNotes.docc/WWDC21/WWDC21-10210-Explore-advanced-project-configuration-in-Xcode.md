# Explore advanced project configuration in Xcode

Working with more complex Xcode projects? You’ve come to the right place. Discover how you can configure your project to build for multiple Apple platforms, filter content per-platform, create custom build rules and file dependencies, and more. We’ll take you through multi-platform framework targets, detail how to optimize your project and scheme configuration, and show you how to make effective use of configuration settings files. 

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10210", purpose: link, label: "Watch Video (25 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Multi-platform frameworks

Maintaining separate frameworks for each platform come with challenges, such as keeping build settings in sync, and ensuring all of your source files are properly added to your compile sources build phases. Introducing Multi-platform frameworks.

- New in Xcode 13
- allow us to consolidate multiple frameworks into one, providing us simplified target management
- one shared set of build phases
- one shared set of build settings

### How to

1. Open any framework project
2. Go to <kbd>Build Settings</kbd> tab in the project editor
3. Under the <kbd>Architectures</kbd> section, configure the framework to build for all platforms (or the platforms you're interested in) by going to the <kbd>Supported Platforms</kbd> build setting and choosing <kbd>Any Platform</kbd>
4. Under the <kbd>Build Options</kbd> section, make sure that <kbd>Allow Multiplatform Builds</kbd> is set to <kbd>Yes</kbd>

This informs the build system to build this target once for each of its supported platforms, as necessary.

### Platform filters

If some frameworks files should target only one or a few specific platforms, we can add a platform filter to specify that this file should only build for those platforms.

How to: 

1. Go to <kbd>Build Phases</kbd> tab in the project editor
2. Expand the <kbd>Compile Sources</kbd> build phase
3. For each file, set the proper platform filter (by default all files will be used within all platforms)

## Project model and configuration best practices

### Scheme build options

For <kbd>Build Order</kbd>, we recommend selecting <kbd>Dependency Order</kbd>, which will cause targets in your project to build in parallel according to the dependency graph. This can greatly improve multicore build performance and will also get you faster results from continuous integration. In contrast, choosing <kbd>Manual Order</kbd> is deprecated and is not recommended. Using this option will slow down your build and can cause cycle errors when the target order listed in the scheme is inconsistent with your project's dependencies. 

For <kbd>Find Implicit Dependencies</kbd>. Checking this option allows Xcode to automatically add dependencies between targets based on the information in your project, such as linker flags in build settings and the names of linked libraries in build phases. This can be especially useful when the related targets are in different projects where you can't normally add an explicit target dependency.

### Build Rules & Script Phases

#### Build Rules

If we have a script phase that, independently, uses projects files for example for generating code, we should use a build rule instead.

1. Go to <kbd>Build Rules</kbd> tab in the project editor
2. Add a new build rule
3. Add the file pattern corresponding to our files we want this rule to process (e.g., `*.recipe`), we don't need to add any additional inputs to the build rule, because it will automatically get each input file it processes as an input (thanks to this pattern matching).
4. We need to tell the build system the path of the output file that the rule will produce for each file it processes. Add a new output file in the <kbd>Output Files</kbd> section. For example `$(DERIVED_FILE_DIR)/$(INPUT_FILE_BASE).compiledrecipe`. 
  - It's best practice to write generated files under `DERIVED_FILE_DIR`, since this will point to an appropriate location managed by the build system. You should avoid generating output files under the source root. This can interfere with source control and lead to conflicts when running multiple builds simultaneously. 

5. Lastly, we need to move over the script body into the build rule. For Example:

```shell
"$SCROOT/Scripts/gen-code.sh" "$SCRIPT_INPUT_FILE" "$SCRIPT_OUTPUT_FILE_0"
```

- Remember that rules run once for each input they process.
- `$SCRIPT_INPUT_FILE` corresponds to the absolute file path of the current input file being processed
- `$SCRIPT_OUTPUT_FILE_0` refers to the output file path we entered in the <kbd>Output Files</kbd> section

6. If the generated code is architecture-independent, uncheck the <kbd>Run once per architecture</kbd> option in the build rule
7. In order for the rule to work, make sure that your rule input files are listed among the <kbd>Compiled Sources</kbd> in the <kbd>Build Phases</kbd> tab of your project

#### Script Phases

When a script phase has no input and output dependencies specified, this might cause build tasks to run in the wrong order and slows down the build because Xcode has to be more conservative with respect to running other tasks in parallel, as it doesn't know what files the script phase may be using. 

It's important to add input and output dependencies to ensure the work performed by script phases is done in the correct order relative to other tasks in the build. 

If a script phase has a large number of inputs, instead of entering these one by one, I can use an <kbd>.xcfilelist</kbd> to manage this list of inputs via an external file. Create one such files: <kbd>File > New File</kbd>, and choose <kbd>Build Phase File List</kbd> under the <kbd>Other section</kbd>. 

Important: the build system requires that only one task in the entire build may produce the output at a given path. With multi-platform targets, the script will run for each platform, therefore if the output path is platform-independent, this will fail the build as each platform build will try to write in the same path. To fix this we have two main ways: 

- (easy, but wasteful) make sure the output path is platform dependent (e.g., by using `DERIVED_FILE_DIR`). This causes our script to run for each platform, but the output will be in different places.
- use aggregate target(s)

#### Environment

The build settings of the target are also made available to the script/build rule phase environment.

There are some crucial environment variables provided for you by the script phase:

```shell
// These environment variables are available in script phases:

SCRIPT_INPUT_FILE_COUNT // This specifies the number of paths from the Input Files table.
SCRIPT_INPUT_FILE_n // This specifies the absolute path of the nth file from the Input Files table, with build settings expanded.

SCRIPT_INPUT_FILE_LIST_COUNT // This specifies the number of input file lists.
SCRIPT_INPUT_FILE_LIST_n // This specifies the absolute path of the nth "resolved" input file list with contained paths made absolute, build settings expanded, and comments removed.

SCRIPT_OUTPUT_FILE_COUNT // This specifies the number of paths from the Output Files table.
SCRIPT_OUTPUT_FILE_n // This specifies the absolute path of the nth file from the Output Files table, with build settings expanded.

SCRIPT_OUTPUT_FILE_LIST_COUNT // This specifies the number of output file lists.
SCRIPT_OUTPUT_FILE_LIST_n // This specifies the absolute path of the nth "resolved" output file list with contained paths made absolute, build settings expanded, and comments removed.

* n in the above examples refers to a 0-based index.
```

And here are the equivalent for build rules:

```shell
// These environment variables are available in build rules:

SCRIPT_INPUT_FILE // This specifies the absolute path of the main input file being processed by the rule.

OTHER_INPUT_FILE_FLAGS // This specifies custom command line flags defined for the input file in the Compile Sources build phase.

SCRIPT_INPUT_FILE_COUNT // This specifies the number of paths from the Input Files table.
SCRIPT_INPUT_FILE_n // This specifies the absolute path of the nth file from the Input Files table, with build settings expanded.

SCRIPT_OUTPUT_FILE_COUNT // This specifies the number of paths from the Output Files table.
SCRIPT_OUTPUT_FILE_n // This specifies the absolute path of the nth file from the Output Files table, with build settings expanded.

SCRIPT_HEADER_VISIBILITY // This is set to "public" or "private" if the input file being processed is a header file in a Headers build phase, and its Header Visibility is set to one of those values.

HEADER_OUTPUT_DIR // This specifies the output directory to which the input file should be copied, if the input file being processed is a header file in a Headers build phase.

* n in the above examples refers to a 0-based index.
```

## Build settings deep dive

So what is a build setting? It is a property you can apply to Xcode targets to configure aspects of how they are built. 

Xcode provides two main mechanisms for configuring build settings:

1. through the build settings editor
2. through a configuration settings file or an <kbd>.xcconfig</kbd> file

### Levels

Build settings are defined at multiple levels, which can be visualized by clicking on the <kbd>Levels</kbd> filter in the build settings editor. 

Each column represents a different level a build setting can be defined in, and they are evaluated from the right to the left. Starting from the lowest (rightmost) level, there are:

1. the default value, which is defined by the currently selected SDK
2. the project level configuration settings file
3. the project level settings from the Xcode project file
4. target settings defined in a configuration settings file
5. the target level settings defined in your Xcode project file
6. the resolved value of the build setting.

### Configuration settings files (<kbd>.xcconfig</kbd>)

Configuration settings file format documentation can be found [here](https://help.apple.com/xcode/#/dev745c5c974).

Advantages:

1. Better source control management
2. Sharing settings across targets or configurations
3. Advanced composition of build settings
4. Ability to include additional configuration settings files based on your development or test environment

At its most basic level, a build setting is made up of a name, an assignment operator, and a value:

```
// Comments are supported, too!
MY_BUILD_SETTING_NAME = "A build setting value"
```

You can narrow the value of a build setting using the conditional syntax. Conditional settings are defined using square brackets. Some of the supported conditions include configuration, architecture, and SDK:

```
MY_BUILD_SETTING_NAME[config=Debug] = -debug_flag

MY_BUILD_SETTING_NAME[arch=arm64] = -arm64_only

// wildcards can be used for matching purposes:
MY_BUILD_SETTING_NAME[sdk=iphone*] = -ios_only
```

A build setting can be set to the value of another build setting by using the dollar-parens syntax: 

```
MY_OTHER_BUILD_SETTING = YES
MY_BUILD_SETTING_NAME = $(MY_OTHER_BUILD_SETTING)

// Multiple values can be evaluated here as well:
MORE_SETTINGS = $(SETTING) $(ANOTHER_SETTING)

// Existing values for a build setting can be used with the $(inherited) value:
APPEND_TO_EXISTING_SETTINGS = $(inherited) -more_flag

// Alternatively, you could obtain the same result with the following:
APPEND_TO_EXISTING_SETTINGS = $(inherited) -more_flag
```

Another use of the build setting evaluation syntax is to compose build settings together from a set of other build settings:

```
IS_BUILD_SETTING_ENABLED = NO
MY_BUILD_SETTING_NO = -use_this_one
MY_BUILD_SETTING_YES = -use_this_instead

// Because build setting evaluation happens inside-out, this will evaluate to $(MY_BUILD_SETTING_NO) first, and then to -use_this_one
MY_BUILD_SETTING = $(MY_BUILD_SETTING_$(IS_BUILD_SETTING_ENABLED))
```


When evaluating a build setting, there are a set of operators you can use to provide some basic transformations of your value.  
The three classifications of operators are: 

1. String operators

| Operator | Value | Transformation |
| --- | --- | --- |
| `$(MY_SETTING:quote)` | characters will be "escaped" | characters\ will\ be\ \"escaped\" |
| `$(MY SETTING:lower)` | All Caps Will Be lowered | all caps will be lowered |
| `$(MY_SETTING:upper)` | make it RAIN | MAKE IT RAIN |
| `$(MY_SETTING:identifier)` | valid c99 identifier | valid_c99_identifier |
| `$(MY_SETTING:c99extidentifier)` | valid c99 ext identifier | valid_c99_ext_identifier |
| `$(MY_SETTING:rfc1034identifier)` | A valid RFC1034 identifier | A-valid-RFC1034-identifier |

2. Path operators

| Operator | Value | Transformation |
| --- | --- | --- |
| `$(MY_PATH:dir)` | `/projects/goodies/src/winner.swift` | `/projects/goodies/src/` |
| `$(MY_PATH:file)` | `/projects/goodies/src/winner.swift` | `winner.swift` |
| `$(MY_PATH:base)` | `/projects/goodies/src/winner.swift` | `winner` |
| `$(MY_PATH:suffix)` | `/projects/goodies/src/winner.swift` | `swift` |
| `$(MY_PATH:standardizepath)` | `/projects/goodies/src/oops//winner.swift` | `/projects/goodies/src/winner.swift` |

3. Replacement operators

| Operator | Value | Transformation |
| --- | --- | --- |
| `$(MY_PATH:dir=/tmp)` | `/projects/goodies/src/winner.swift` | `/tmp/winner.swift` |
| `$(MY_PATH:file=/better.swift)` | `/projects/goodies/src/winner.swift` | `better.swift` |
| `$(MY PATH:base=another)` | `/projects/goodies/src/winner.swift` | `/projects/goodies/src/another.swift` |
| `$(MY_PATH:suffix=m)` | `/projects/goodies/src/winner.swift` | `/projects/goodies/src/winner.m` |
| `$(MY_PATH:default=YES)` | no value | `YES` |
| `$(MY_PATH:default=YES)` | has value | existing value |

> the default operator provides the replacement value if the build setting is empty, otherwise it uses the existing value of the build setting

Including other configuration settings files:

```
// Require the included configuration settings file to exist on disk. 
// This will stop the build process if the specified configuration is not found.
#include "path/to/file.xcconfig"

// Allowing for optionally including a configuration settings file:
#include? "path/to/file.xcconfig"
```

> the `include` path is relative from the location of your Xcode project file