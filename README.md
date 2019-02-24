# FSH

FSH (**F**# **Sh**ell) is a 'shell', like CMD, Powershell or Bash, entirely written in F#.

## Basic Interactions

In addition to normal shell features (folder navigation, creation, deletion, echo etc.), FSH also supports piping and F# interactive. You can run code to declare F# functions, then use those functions as part of a piping operation with the other builtin commands. You can also just declare anonymous expressions for piping.

To demonstrate what this means, using FSH, you could type a line like:

	echo "hello world" |> (fun s -> s.ToUpper()) >> result.txt

Which would create a text file in the current directory called result.txt, containing the text `HELLO WORLD`

For something more advanced, you could implement a simple 'grep' command (grep is not built in to FSH by default):

	(let grep (s:string) arr = 
		arr |> Array.filter (fun line -> line.Contains(s)))

(**Note** that shift+enter will go to a new line, useful for code. Tabbing (four spaces) has to be entered manually. Also, by piping arr on the second line, the line parameter does not need a type annotation.)

Then use this like:

	ls |> (grep ".dll")

## Installing and Running

FSH has been built using **.NET Core 2.2**. To build, you will need to install this SDK from [here](https://dotnet.microsoft.com/download/dotnet-core/2.2).

It has been tested in Linux (via Ubuntu 18 under WSL) and on Max OSX, without issues.

To run on the command line, navigate to the **/src** directory and use the command `dotnet run`

It was built with Visual Studio 2017 Community (with occasional work done in VS Code) so instead of the CLI, you could open the **FSH.sln** solution file and run using F5 in VS.

There are also some XUnit/FsUnit tests defined under /test, there for testing various input scenarios through the parts/tokenisation functions. Run them (if you wish) via `dotnet test`

**Important**: This was built in a month, and shells are a lot more complicated than they look. There *are* bugs, but *hopefully* during casual use you won't find them :)

## "Builtins"

Any shell has a number of commands that you can run, aside from things like external processes or in the case of FSH, code. FSH supports a limited set of these, all with basic but effective functionality. To get a list of these type **help** or **?**. The most heavily used are:

- **cd [path]**: change the current directory. E.g. `cd /` or `cd ..` to go to root or one directory up.
- **echo [text]**: prints the arguments out as text - quite useful for piping
- **cat [filename]** reads a file and prints out its output - also useful for piping
- **>> [filename]** prints out the piped in content into a file.

There are almost a dozen further commands, like **rm**, **mkdir**, **env** etc.

All builtins are defined in [Builtins.fs](/src/Builtins.fs)

## Code expressions

Code expressions are written by wrapping code with `(` and `)`. Only the outer most parentheses are parsed as wrappers, so for example `(fun (s:string) -> s.ToUpper())` is perfectly fine. 

If a code expression is the first expression on a line, then it is treated as a **FSI Interaction**. Otherwise it is treated as a **FSI Expression**. What is the difference?

- A FSI Interaction can be any valid F# code. `let x = 10`, `50 * 5`, `let grep (s: string) arr = arr |> Array.filter (fun line -> line.Contains(s))` are all valid interactions.
- A FSI Expression must be F# that evaluates to a value. `let x = 10` is not a valid expression, but `let x = 10 in x` *is* valid as it will evaluate to 10. Likewise, defining functions like `let grep ...` above is not valid unless it is immediately used in a `in` expression, like `let grep... in grep ".dll"` (which also locally scope grep, making it not usable again without being redefined).

### Piped values

Because expressions are only run when there is a value to be piped into them, they have the additional contraint that they must support a parameter that matches the piped value (either a string or a string array). The reason is that in `echo "test" |> (fun s -> s.ToUpper())`, the code that is passed to the hidden FSI process is actually `let piped = "test" in piped |> (fun s -> s.ToUpper())`

Normally the piped value will be a string, as above. However if the prior result contains line breaks, it will be split into an array. E.g. ls or dir, if used on a directory with more than one file, will produce a piped string array for code expressions to use.

An expression can return either a value or a string array. If the latter, than it will be concatenated with line breaks for output printing. Otherwise it is converted to a string.

### Interactions and 'it'

By default, FSI will evaluate an interaction and return `unit` if successful. 
In order to make interactions useful as the first token of a piped expression, after an interaction is evaluated FSH will attempt to evaluate the expression 'it' and send its value on to the next token (or console out, if the last expression).

If this evaluation fails, then an empty string is passed along. Otherwise, if found, then it is *set* to "", before the value is passed along. This is so that it is cleared in case the use of later interaction doesnt overwrite it.

Code is run in [Interactive.fs](/src/Interactive.fs), which wraps FSI. For further details please follow the code in there.

## Why?

This has been developed for the 2019 F# Applied Competition, as an educational project, over the course of about a month. 

The idea came from PowerShell, which as a developer who works primarily on windows machines, is my default shell. However, PowerShell syntax is very verbose, especially when using .NET code in line: a shell with a simpler, more bash or cmd-like syntax combined with the light syntax and type inferrence of F# seemed like a good thing to make into a proof-of-concept.

As it stands, FSH is not really intended to be used in anger: while it supports a host of builtins, and is quite extensible with its FSI integration, it is missing a lot of features like, for example, numerous flags on the default builtins (ls/dir just list the local directory, but don't for example, have any flags to provide more info or multi page support etc).

If someone wanted to take this, and extend it, they are free to do so: its under MIT. But ultimately, FSH serves as an educational example of:

- Creating a shell, able to run its own commands and external executables.
- Various folder and file manipulation techniques.
- A custom ReadLine method, that supports advanced features like line shifting and history (defined in [LineReader.fs](/src/LineReader.fs))
- String tokensisation and rendering (defined in [Terminal.fs](/src/Terminal.fs))
- Integrating F# Interactive into a project in a useful way.

All code files are helpfully commented. Start with [Program.fs](/src/Program.fs) and follow its structure from there.

I hope it is useful to readers on the above topics :)