# goroutine-inspect

An interactive tool to analyze Golang goroutine dump.

## Build and Run

```bash
go get github.com/linuxerwang/goroutine-inspect
$GOPATH/bin/goroutine-inspect
```

## Workspace

Workspace is the place to hold imported goroutine dumps. Instructions are
provided to maintain these dumps.

In the interactive shell, two kinds of instructions can be issued: commands
and statements.

## Commands

At present, only 6 commands are supported.

| command | function                          |
| ------- | --------------------------------- |
| cd      | Change current working directory. |
| clear   | Clear the workspace.              |
| help    | Show this help.                   |
| ls      | Show files in current directory.  |
| whos    | Show all varaibles in workspace.  |

## Statements

### Load Goroutine Dump From Files

Load the dump and assign to a variable:

```bash
>> original = load("pprof-goroutines-20170510-170245.dump")
>> whos
original
```

### Show the Summary of a Dump Var

Simply type the variable name:

```bash
>> original
# of goroutines: 2217

        running: 1
        IO wait: 533
        syscall: 2
   chan receive: 50
         select: 1504
       runnable: 38
     semacquire: 85
      chan send: 4

```

### Copy a Dump Var

To copy the whole dump, simply assign it to a different var:

```bash
>> copy1 = original
>> whos
copy        original
```

It's equivalent to using a copy() function:

```bash
>> copy2 = original.copy()
>> whos
copy        copy1        copy2        original
```

The copy() function allows passing a conditional so that only those meeting
the cariteria will be copied:

```bash
>> copy3 = original.copy("id>900 && id<2000")
```

### Modify the Dump Goroutine Items

Function delete() accepts a conditional to delete goroutine items in a dump
var. Function keep() do the reversed conditional.

```bash
>> copy
# of goroutines: 2217

        running: 1
        IO wait: 533
        syscall: 2
   chan receive: 50
         select: 1504
       runnable: 38
     semacquire: 85
      chan send: 4

>> copy.delete("id>100 && id<1000")
Deleted 118 goroutines, kept 2099.
>> copy.keep("id>200")
Deleted 12 goroutines, kept 2087.
>> copy
# of goroutines: 2087

        running: 1
         select: 1411
        IO wait: 500
     semacquire: 85
       runnable: 37
   chan receive: 49
      chan send: 4

```

### Display Goroutine Dump Items

Function show() displays goroutine dump items with optional offset and limit.
The default offset is 0, and default limit is 10.

```bash
>> original.show() # offset 0, limit 10

goroutine 1803 [select, 10 minutes]:
google.golang.org/grpc/transport.(*http2Server).keepalive(0xc420e59ce0)
        google.golang.org/grpc/transport/http2_server.go:919 +0x488
created by google.golang.org/grpc/transport.newHTTP2Server
        google.golang.org/grpc/transport/http2_server.go:226 +0x97c

...
...

>> original.show(15) # offset 15, limit 10

goroutine 6455709 [running]:
runtime/pprof.writeGoroutineStacks(0xe9a080, 0xc4216f0088, 0x1d, 0x40)
        go1.8.1.linux-amd64/src/runtime/pprof/pprof.go:603 +0x79
runtime/pprof.writeGoroutine(0xe9a080, 0xc4216f0088, 0x2, 0x1d, 0xc4217cede0)
        go1.8.1.linux-amd64/src/runtime/pprof/pprof.go:592 +0x44
runtime/pprof.(*Profile).WriteTo(0xed3780, 0xe9a080, 0xc4216f0088, 0x2, 0xc4217cef80, 0x1)
        go1.8.1.linux-amd64/src/runtime/pprof/pprof.go:302 +0x3b5
www.test.com/bagel/runtime.dumpToFile(0xed0f0ba5e, 0xae05027, 0xee1780, 0xc425bd2060, 0x5, 0x5)
        www.test.com/bagel/runtime/dump.go:58 +0x3f3
created by www.test.com/bagel/runtime.EnableGoroutineDump.func1
        www.test.com/bagel/runtime/dump.go:30 +0x2d6

...
...

>> original.show(15, 1) # offset 15, limit 1

goroutine 6455709 [running]:
runtime/pprof.writeGoroutineStacks(0xe9a080, 0xc4216f0088, 0x1d, 0x40)
        go1.8.1.linux-amd64/src/runtime/pprof/pprof.go:603 +0x79
runtime/pprof.writeGoroutine(0xe9a080, 0xc4216f0088, 0x2, 0x1d, 0xc4217cede0)
        go1.8.1.linux-amd64/src/runtime/pprof/pprof.go:592 +0x44
runtime/pprof.(*Profile).WriteTo(0xed3780, 0xe9a080, 0xc4216f0088, 0x2, 0xc4217cef80, 0x1)
        go1.8.1.linux-amd64/src/runtime/pprof/pprof.go:302 +0x3b5
www.test.com/bagel/runtime.dumpToFile(0xed0f0ba5e, 0xae05027, 0xee1780, 0xc425bd2060, 0x5, 0x5)
        www.test.com/bagel/runtime/dump.go:58 +0x3f3
created by www.test.com/bagel/runtime.EnableGoroutineDump.func1
        www.test.com/bagel/runtime/dump.go:30 +0x2d6
```

### Search Goroutine Dump Items

Similar to show(), but with a conditional to only show items meeting certain
criteria:

```bash
>> original.search("id < 2000", 15, 1) # offset 15, limit 1

goroutine 6455896 [select]:
net.(*netFD).connect.func2(0xea1980, 0xc424bca540, 0xc422c1af50, 0xc424bca600, 0xc424bca5a0)
        go1.8.1.linux-amd64/src/net/fd_unix.go:133 +0x1d5
created by net.(*netFD).connect
        go1.8.1.linux-amd64/src/net/fd_unix.go:144 +0x239
```

## Properties of a Goroutine Dump Item

Each dump item has 5 properties which can be used in conditionals:

| property | type    | meaning                                             |
| -------- | ------- | --------------------------------------------------- |
| id       | integer | The goroutine ID.                                   |
| duration | integer | The waiting duration of a goroutine.                |
| lines    | integer | The number of lines of the goroutine's stack trace. |
| state    | string  | The running state of the goroutine.                 |
| trace    | string  | The concatenated text of the goroutine stack trace. |
