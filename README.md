# Multitemplate

[![Run Tests](https://github.com/caris-events-fork/multitemplate/actions/workflows/go.yml/badge.svg)](https://github.com/caris-events-fork/multitemplate/actions/workflows/go.yml)
[![codecov](https://codecov.io/gh/gin-contrib/multitemplate/branch/master/graph/badge.svg)](https://codecov.io/gh/gin-contrib/multitemplate)
[![Go Report Card](https://goreportcard.com/badge/github.com/caris-events-fork/multitemplate)](https://goreportcard.com/report/github.com/caris-events-fork/multitemplate)
[![GoDoc](https://godoc.org/github.com/caris-events-fork/multitemplate?status.svg)](https://godoc.org/github.com/caris-events-fork/multitemplate)

This is a custom HTML render to support multi templates, ie. more than one `*template.Template`.

## Usage

### Start using it

Download and install it:

```sh
go get github.com/caris-events-fork/multitemplate
```

Import it in your code:

```go
import "github.com/caris-events-fork/multitemplate"
```

### Simple example

See [example/simple/example.go](example/simple/example.go)

```go
package main

import (
  "github.com/caris-events-fork/multitemplate"
  "github.com/gin-gonic/gin"
)

func createMyRender() multitemplate.Renderer {
  r := multitemplate.NewRenderer()
  r.AddFromFiles("index", "templates/base.html", "templates/index.html")
  r.AddFromFiles("article", "templates/base.html", "templates/index.html", "templates/article.html")
  return r
}

func main() {
  router := gin.Default()
  router.HTMLRender = createMyRender()
  router.GET("/", func(c *gin.Context) {
    c.HTML(200, "index", gin.H{
      "title": "Html5 Template Engine",
    })
  })
  router.GET("/article", func(c *gin.Context) {
    c.HTML(200, "article", gin.H{
      "title": "Html5 Article Engine",
    })
  })
  router.Run(":8080")
}
```

### Advanced example

[Approximating html/template Inheritance](https://elithrar.github.io/article/approximating-html-template-inheritance/)

See [example/advanced/example.go](example/advanced/example.go)

```go
package main

import (
  "path/filepath"

  "github.com/caris-events-fork/multitemplate"
  "github.com/gin-gonic/gin"
)

func main() {
  router := gin.Default()
  router.HTMLRender = loadTemplates("./templates")
  router.GET("/", func(c *gin.Context) {
    c.HTML(200, "index.html", gin.H{
      "title": "Welcome!",
    })
  })
  router.GET("/article", func(c *gin.Context) {
    c.HTML(200, "article.html", gin.H{
      "title": "Html5 Article Engine",
    })
  })

  router.Run(":8080")
}

func loadTemplates(templatesDir string) multitemplate.Renderer {
  r := multitemplate.NewRenderer()

  layouts, err := filepath.Glob(templatesDir + "/layouts/*.html")
  if err != nil {
    panic(err.Error())
  }

  includes, err := filepath.Glob(templatesDir + "/includes/*.html")
  if err != nil {
    panic(err.Error())
  }

  // Generate our templates map from our layouts/ and includes/ directories
  for _, include := range includes {
    layoutCopy := make([]string, len(layouts))
    copy(layoutCopy, layouts)
    files := append(layoutCopy, include)
    r.AddFromFiles(filepath.Base(include), files...)
  }
  return r
}
```
