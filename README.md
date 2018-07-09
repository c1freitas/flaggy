<p align="center">

<img src="/logo.png" />
<br />
<a href="https://goreportcard.com/report/github.com/integrii/flaggy"><img src="https://goreportcard.com/badge/github.com/integrii/flaggy"></a>
<a href="https://travis-ci.org/integrii/flaggy"><img src="https://travis-ci.org/integrii/flaggy.svg?branch=master"></a>
<a href="http://godoc.org/github.com/integrii/flaggy"><img src="https://camo.githubusercontent.com/d48cccd1ce67ddf8ba7fc356ec1087f3f7aa6d12/68747470733a2f2f676f646f632e6f72672f6769746875622e636f6d2f6c696c65696f2f6c696c653f7374617475732e737667"></a>
<img src="https://img.shields.io/badge/license-Unlicense-blue.svg">
</p>

Sensible and _fast_ command-line flag parsing with excellent support for **subcommands** and **positional values**. Flags can be at any position.  Flaggy has no required project or package layout like [Cobra requires](https://github.com/spf13/cobra/issues/641), and **no external dependencies**!

Check out the [godoc](http://godoc.org/github.com/integrii/flaggy), [examples directory](https://github.com/integrii/flaggy/tree/master/examples), and [examples in this readme](https://github.com/integrii/flaggy#super-simple-example) to get started quickly.

# Installation

`go get -u github.com/integrii/flaggy`

# Key Features

- Very easy to use ([see examples below](https://github.com/integrii/flaggy#super-simple-example))
- 35 different flag types supported
- Any flag can be at at any position
- Pretty and readable help output by default
- Positional subcommands
- Positional parameters
- Suggested subcommands when a subcommand is typo'd
- Nested subcommands
- Both global and subcommand specific flags
- Both global and subcommand specific positional parameters
- Customizable help templates for both the global command and subcommands
- Customizable appended/prepended help messages for both the global command and subcommands
- Simple function that displays help followed by a custom message string
- Flags and subcommands may have both a short and long name
- Unlimited trailing arguments after a `--`
- Flags can use a single dash or double dash (`--flag`, `-flag`, `-f`, `--f`)
- Flags can have `=` assignment operators, or use a space (`--flag=value`, `--flag value`)
- Flags support single quote globs with spaces (`--flag 'this is all one value'`)
- Flags of slice types can be passed multiple times (`-f one -f two -f three`)
- Optional but default version output with `-v` or `--version`
- Optional but default help output with `-h` or `--help`
- Optional but default help output when any invalid or unknown parameter is passed
- It's _fast_. All flag and subcommand parsing takes less than `1ms` in most programs.


# Example Help Output

```
testCommand - Description goes here.  Get more information at http://flaggy.
This is a prepend for help

  Usage:
    testCommand [subcommandA|subcommandB|subcommandC] [testPositionalA] [testPositionalB]

  Positional Variables:
    testPositionalA - (Required) Test positional A does some things with a positional value. (default: defaultValue)
    testPositionalB - Test positional B does some less than serious things with a positional value.

  Subcommands:
    subcommandA (a) - Subcommand A is a command that does stuff
    subcommandB (b) - Subcommand B is a command that does other stuff
    subcommandC (c) - Subcommand C is a command that does SERIOUS stuff

  Flags:
    -s --stringFlag  This is a test string flag that does some stringy string stuff.
    -i --intFlg  This is a test int flag that does some interesting int stuff. (default: 5)
    -b --boolFlag  This is a test bool flag that does some booly bool stuff. (default: true)
    -d --durationFlag  This is a test duration flag that does some untimely stuff. (default: 1h23s)

This is an append for help
This is a help addon message
```

# Supported Flag Types:

- string
- []string
- bool
- []bool
- time.Duration
- []time.Duration
- float32
- []float32
- float64
- []float64
- uint
- uint64
- []uint64
- uint32
- []uint32
- uint16
- []uint16
- uint8
- []uint8
- []byte
- int
- []int
- int64
- []int64
- int32
- []int32
- int16
- []int16
- int8
- []int8
- net.IP
- []net.IP
- net.HardwareAddr
- []net.HardwareAddr
- net.IPMask
- []net.IPMask


# Super Simple Example

`./yourApp -f test`

```go
// Declare variables and their defaults
var stringFlag = "defaultValue"

// Add a flag
flaggy.String(&stringFlag, "f", "flag", "A test string flag")

// Parse the flag
flaggy.Parse()

// Use the flag
print(stringFlag)
```


# Example with Subcommand

`./yourApp subcommandExample -f test`

```go
// Declare variables and their defaults
var stringFlag = "defaultValue"

// Create the subcommand
subcommand := flaggy.NewSubcommand("subcommandExample")

// Add a flag to the subcommand
subcommand.String(&stringFlag, "f", "flag", "A test string flag")

// Add the subcommand to the parser at position 1
flaggy.AttachSubcommand(subcommand, 1)

// Parse the subcommand and all flags
flaggy.Parse()

// Use the flag
print(stringFlag)
```

# Example with Nested Subcommands, Various Flags and Trailing Arguments

`./yourApp subcommandExample --flag=5 nestedSubcommand -t test -y -- trailingArg`

```go
// Declare variables and their defaults
var stringFlagF = "defaultValueF"
var intFlagT = 3
var boolFlagB bool

// Create the subcommand
subcommandExample := flaggy.NewSubcommand("subcommandExample")
nestedSubcommand := flaggy.NewSubcommand("nestedSubcommand")

// Add a flag to the subcommand
subcommandExample.String(&stringFlagF, "t", "testFlag", "A test string flag")

nestedSubcommand.Int(&intFlagT, "f", "flag", "A test int flag")

// add a global bool flag for fun
flaggy.Bool(&boolFlagB, "y", "yes", "A sample boolean flag")

//  the nested subcommand to the parent subcommand at position 1
subcommandExample.AttachSubcommand(nestedSubcommand, 1)
//  the base subcommand to the parser at position 1
flaggy.AttachSubcommand(subcommandExample, 1)

// Parse the subcommand and all flags
flaggy.Parse()

// Use the flags and trailing arguments
print(stringFlagF)
print(intFlagT)
print(boolFlagB)
print(flaggy.TrailingArguments[0])
```


# Other Tricks

- Set a name and description for your program.
- Disable help display when flaggy has an error parsing or validating user input.
- Prepend a message to flag help output.

```go
package main

import "github.com/integrii/flaggy"

// make a variable for the version which will be set at build time
var version = "unknown"

func init() {
  // Set your program's name and description, if you want to.
  // This shows when you run help
  flaggy.SetName("Test Program")
  flaggy.SetDescription("A little example program")

  // you can disable various things by changing bools on the default parser (or your own parser if you have created one)
  flaggy.DefaultParser.ShowHelpOnUnexpected = false

  // you can set a help prepend or append on the default parser
  flaggy.DefaultParser.AdditionalHelpPrepend = "http://github.com/integrii/flaggy"

  // set the version and parse
  flaggy.SetVersion(version)
  flaggy.Parse()
}
func main(){}
```

##### Set Flaggy Version at Build Time

```bash
# build your app and set the version string
$ go build -ldflags='-X main.version=1.0.3-a3db3'
$ ./yourApp version
Version: 1.0.3-a3db3
$ ./yourApp --help
Test Program - A little example program
http://github.com/integrii/flaggy
```
