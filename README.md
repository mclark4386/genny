<p align="center"><img src="https://github.com/gobuffalo/buffalo/blob/master/logo.svg" width="360"></p>

<p align="center">
<a href="https://godoc.org/github.com/gobuffalo/genny"><img src="https://godoc.org/github.com/gobuffalo/genny?status.svg" alt="GoDoc" /></a>
<a href="https://travis-ci.org/gobuffalo/genny"><img src="https://travis-ci.org/gobuffalo/genny.svg?branch=master" alt="Build Status" /></a>
<a href="https://goreportcard.com/report/github.com/gobuffalo/genny"><img src="https://goreportcard.com/badge/github.com/gobuffalo/genny" alt="Go Report Card" /></a>
</p>

# Genny [EXPERIMENTAL]

**EXPERIMENTAL** - APIs can change without notice. You've been warned.

## What Is Genny?

Genny is a _framework_ for writing modular generators, it however, doesn't actually generate anything. It just makes it easier for you to. :)

## The `Generator` Interface

Genny was inspired by the [`context`](https://golang.org/pkg/context/) design and foregoes any configuration for a "wrapping" pattern. See the example at the bottom of the file or in the `./examples` folder.

```go
type Generator interface {
	Context() context.Context
	Run() error
	Parent() Generator
	Logger() Logger
}
```

## Documentation

For right now the [GoDoc](https://godoc.org/github.com/gobuffalo/genny) and the source/tests are best documentation as the APIs are currently in flux.

## Example

```go
package main

import (
	"log"
	"os/exec"
	"strings"

	"text/template"

	"github.com/gobuffalo/genny"
	"github.com/gobuffalo/genny/movinglater/gotools"
	"github.com/gobuffalo/genny/movinglater/plushgen"
	"github.com/gobuffalo/plush"
)

func main() {
	// setup a new generator:
	g := genny.Background()
	// process the previously declared file using the plush package:
	ctx := plush.NewContextWithContext(g.Context())
	ctx.Set("name", "Bates")
	g = plushgen.WithPlush(g, ctx)

	// process the previously declared file using the text/template package:
	g = gotools.WithTemplate(g, "mark", template.FuncMap{})

	// wrap in a "dry runner" so files and commands are echoed to the screen, but not executed:
	g = genny.DryRun(g)

	// wrap in a "wet runner" so files and commands will be executed
	// g = genny.WetRun(g)

	// wrap the previous generator with a command to run:
	g = genny.WithCmd(g, exec.CommandContext(g.Context(), "echo", "hello from the echo command!"))

	// wrap the previous generator with a file: (path to write to, reader to read from)
	g = genny.WithFileFromReader(g, "examples/output/foo.txt", strings.NewReader("Hello {{.}} <%= name %>"))

	// install a package with go get
	g = gotools.WithGoGet(g, "github.com/gobuffalo/envy", "-v")

	// run another command
	g = genny.WithCmd(g, exec.CommandContext(g.Context(), "echo", "almost finished"))

	// add the contents of a packr box to the generators
	// box := packr.NewBox("../")
	// g, err = packrgen.WithBox(g, box, func(gg genny.Generator, f genny.File) genny.Generator {
	// 	return genny.WithFileFromReader(gg, filepath.Join("examples", "output", f.Name()), f)
	// })
	// if err != nil {
	// 	log.Fatal(err)
	// }

	// add another file
	g = genny.WithFileFromReader(g, "examples/output/baz/bar.txt", strings.NewReader("plain text"))

	// try to get the same package, but it won't run 2x
	g = gotools.WithGoGet(g, "github.com/gobuffalo/envy", "-v")
	g = gotools.WithGoGet(g, "github.com/gobuffalo/plush", "-u", "-v")

	// actually run the generators:
	err := genny.Run(g)
	if err != nil {
		log.Fatal(err)
	}
}
```
