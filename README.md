# picocli - a mighty tiny command line interface

Annotation-based Java command line parser. Usage help with ANSI colors. Autocomplete. Easily included as source to avoid adding dependencies. Nested subcommands.

|Now in BETA: [command line autocompletion](http://picocli.info/1.0.0-SNAPSHOT/autocomplete.html)|
|----|
|Autocomplete is work in progress. Comments, bug reports, pull requests welcome!
![Autocompletion demo animation](docs/images/picocli-autocompletion-demo.gif?raw=true)|

A Java command line parsing framework in a single file, so you can include it _in source form_.
This lets users run picocli-based applications without requiring picocli as an external dependency.

How it works: annotate your class and picocli initializes it from the command line arguments,
converting the input to strongly typed data. Supports git-like [subcommands](http://picocli.info/#_subcommands)
(and nested [sub-subcommands](http://picocli.info/#_nested_sub_subcommands)),
any option prefix style, POSIX-style [grouped short options](http://picocli.info/#_short_options),
custom [type converters](http://picocli.info/#_custom_type_converters) and more.

Distinguishes between [named options](http://picocli.info/#_options) and
[positional parameters](http://picocli.info/#_positional_parameters) and allows _both_ to be 
[strongly typed](http://picocli.info/#_strongly_typed_everything).
[Multi-valued fields](http://picocli.info/#_multiple_values) can specify 
an exact number of parameters or a [range](http://picocli.info/#_arity) (e.g., `0..*`, `1..2`).

Generates polished and easily tailored [usage help](http://picocli.info/#_usage_help)
and  [version help](http://picocli.info/#_version_help),
using [ANSI colors](http://picocli.info/#_ansi_colors_and_styles) where possible.
Works with Java 5 or higher (but is designed to facilitate the use of Java 8 lambdas).

<a id="picocli_demo"></a>
![Picocli Demo help message with ANSI colors](docs/images/picocli.Demo.png?raw=true)

* user manual: [http://picocli.info](http://picocli.info)
* [API Javadoc](http://picocli.info/apidocs/)
* [FAQ](https://github.com/remkop/picocli/wiki/FAQ)
* [Releases](https://github.com/remkop/picocli/releases) - latest: 0.9.8
* [![Build Status](https://travis-ci.org/remkop/picocli.svg?branch=master)](https://travis-ci.org/remkop/picocli) 
[![codecov](https://codecov.io/gh/remkop/picocli/branch/master/graph/badge.svg)](https://codecov.io/gh/remkop/picocli)


## Example

Annotate fields with the command line parameter names and description.

```java
import picocli.CommandLine.Option;
import picocli.CommandLine.Parameters;
import java.io.File;

public class Example {
    @Option(names = { "-v", "--verbose" }, description = "Be verbose.")
    private boolean verbose = false;

    @Option(names = { "-h", "--help" }, usageHelp = true,
            description = "Displays this help message and quits.")
    private boolean helpRequested = false;

    @Parameters(arity = "1..*", paramLabel = "FILE", description = "File(s) to process.")
    private File[] inputFiles;
    ...
}
```

Then invoke `CommandLine.populateCommand` with the command line parameters and an object you want to initialize.

```java
String[] args = { "-v", "inputFile1", "inputFile2" };
Example app = CommandLine.populateCommand(new Example(), args);

assert !app.helpRequested;
assert  app.verbose;
assert  app.inputFiles != null && app.inputFiles.length == 2;
```

Invoke `CommandLine.usage` if the user requested help or the input was invalid and a `ParameterException` was thrown.

```java
CommandLine.usage(new Example(), System.out);
```

![Usage help message with ANSI colors](docs/images/ExampleUsageANSI.png?raw=true)

## Usage Help with ANSI Colors and Styles

Colors, styles, headers, footers and section headings are easily customized with annotations.
For example:

![Longer help message with ANSI colors](docs/images/UsageHelpWithStyle.png?raw=true)

See the [source code](https://github.com/remkop/picocli/blob/v0.9.4/src/test/java/picocli/Demo.java#L337). 



## Usage Help API

Picocli annotations offer many ways to customize the usage help message.

If annotations are not sufficient, you can use picocli's [Help API](http://picocli.info/#_usage_help_api) to customize even further.
For example, your application can generate help like this with a custom layout:

![Usage help message with two options per row](docs/images/UsageHelpWithCustomLayout.png?raw=true)

See the [source code](https://github.com/remkop/picocli/blob/master/src/test/java/picocli/CustomLayoutDemo.java#L61).

## API Changes

### [0.9.8](https://github.com/remkop/picocli/releases/tag/v0.9.8)
Non-breaking changes to add better help support and better subcommand support.

### [0.9.7](https://github.com/remkop/picocli/releases/tag/v0.9.7)
Version 0.9.7 has some breaking API changes.

**Better Groovy support**

It was [pointed out](https://github.com/remkop/picocli/issues/135) that Groovy had trouble distinguishing between
the static `parse(Object, String...)` method and the instance method `parse(String...)`.

To address this, the static `parse(Object, String...)` method has been renamed
to `populateCommand(Object, String...)` in  version 0.9.7.

**Nested subcommands**

* Version 0.9.7 adds support for [nested sub-subcommands](https://github.com/remkop/picocli/issues/127)
* `CommandLine::parse` now returns `List<CommandLine>` (was `List<Object>`)
* `CommandLine::getCommands` now returns `Map<String, CommandLine>` (was `Map<String, Object>`)
* renamed method `CommandLine::addCommand` to `addSubcommand`
* renamed method `CommandLine::getCommands` to `getSubcommands`

**Miscellaneous**

Renamed class `Arity` to `Range` since it is not just used for @Option and @Parameters `arity` but also for `index` in positional @Parameters.