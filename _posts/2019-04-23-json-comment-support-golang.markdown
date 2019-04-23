---
layout: post
title: "Json comment support- golang"
date: 2019-04-23 12:12 +0800
categories: tools
published: true
---

Official Json does not support comment, which makes it not so convenient in development, I write a small module to support line comment`//`, multi-line comment `/**/` is not supported yet.

main.go

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
	"path"

	"github.com/wuxiangzhou2010/jsonuncommenter"
)

type seeds struct {
	Seeds []string `json:"seeds"`
	Name  string   `json:"name"`
}

func main() {

	cwd, err := os.Getwd()
	if err != nil {
		panic(err)
	}
	f, err := os.Open(path.Join(cwd, `config.json`))
	if err != nil {
		panic(err)
	}
	defer f.Close()
	newReader := jsonuncommenter.RemoveComment(f)

	jsonParser := json.NewDecoder(newReader)
	var s seeds
	jsonParser.Decode(&s)
	fmt.Printf("%+v", s)
}
```

config.json

```json
{
  "seeds": [
    "https://www.baidu.com" // http下载
    //
    // "https://v.qq.com"
  ],
  "name": "test"
}
```

output

```sh
$ go run main.go
{Seeds:[http://t66y.com/thread0806.php?fid=16] Name:test}
```
