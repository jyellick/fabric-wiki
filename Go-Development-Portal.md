This page is for notes, tips, and suggestions for Go development

# Debugging

The Golang corpus does not include a native debugger.  It is possible to attach GDB to a go program, but the results are often disappointing on account of a lack of insight into the Go runtime (such as goroutines, etc).  The obvious and perhaps idiomatic way to debug a Go application is with the tried and true printf+iterate model.  However, there are several options above and beyond that may make certain problems easier to find.

## Delve

[derekparker/delve](https://github.com/derekparker/delve) is a third party project that offers a comprehensive gdb-like, yet go-aware debugger.  We won't reiterate the description or documentation here, but if you are looking for a way to debug a nasty problem, Delve does a pretty decent job.

### A few helpful tips

You can run your go program with arguments (similar to gdb's "set arg") using the "--" parameter, as per [here](https://github.com/derekparker/delve/wiki/Usage#passing-arguments)

## Forcing a stack trace in a hung program

`kill -ABRT` is a useful way to generate a stack trace of all goroutines, helpful if your program appears hung.
