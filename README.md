[![Go Reference](https://pkg.go.dev/badge/github.com/securitym0nkey/grok.svg)](https://pkg.go.dev/github.com/securitym0nkey/grok)


# grok
A simple library to parse grok patterns with Go.

# Installation
Make sure you have a working Go environment.

```sh
go get github.com/securitym0nkey/grok
```

# Use in your project
```go
import "github.com/securitym0nkey/grok"
```

# Usage
## Available patterns and custom ones
By default this grok package contains only patterns you can see in patterns/grok-patterns file.

When you want to add a custom pattern, use the grok.AddPattern(nameOfPattern, pattern), see the example folder for an example of usage.
You also can load your custom patterns from a file (or folder) using grok.AddPatternsFromPath(path), or PatterndDir configuration.

## Parse all or only named captures
```go
g, _ := grok.New()
values, _  := g.Parse("%{COMMONAPACHELOG}", `127.0.0.1 - - [23/Apr/2014:22:58:32 +0200] "GET /index.php HTTP/1.1" 404 207`)

g, _ = grok.NewWithConfig(&grok.Config{NamedCapturesOnly: true})
values2, _ := g.Parse("%{COMMONAPACHELOG}", `127.0.0.1 - - [23/Apr/2014:22:58:32 +0200] "GET /index.php HTTP/1.1" 404 207`)
```
values is a map with all captured groups
values2 contains only named captures

# Examples
```go
package main

import (
	"fmt"

	"github.com/securitym0nkey/grok"
)

func main() {
	g, _ := grok.New()
	values, _ := g.Parse("%{COMMONAPACHELOG}", `127.0.0.1 - - [23/Apr/2014:22:58:32 +0200] "GET /index.php HTTP/1.1" 404 207`)

	for k, v := range values {
		fmt.Printf("%+15s: %s\n", k, v)
	}
}
```

output:
```
       response: 404
          bytes: 207
       HOSTNAME: 127.0.0.1
       USERNAME: -
       MONTHDAY: 23
        request: /index.php
      BASE10NUM: 207
           IPV6:
           auth: -
      timestamp: 23/Apr/2014:22:58:32 +0200
           verb: GET
    httpversion: 1.1
           TIME: 22:58:32
           HOUR: 22
COMMONAPACHELOG: 127.0.0.1 - - [23/Apr/2014:22:58:32 +0200] "GET /index.php HTTP/1.1" 404 207
       clientip: 127.0.0.1
             IP:
          ident: -
          MONTH: Apr
           YEAR: 2014
         SECOND: 32
            INT: +0200
           IPV4:
         MINUTE: 58
     rawrequest:
```

# Example 2
```go
package main

import (
  "fmt"

  "github.com/securitym0nkey/grok"
)

func main() {
  g, _ := grok.NewWithConfig(&grok.Config{NamedCapturesOnly: true})
  values, _ := g.Parse("%{COMMONAPACHELOG}", `127.0.0.1 - - [23/Apr/2014:22:58:32 +0200] "GET /index.php HTTP/1.1" 404 207`)

  for k, v := range values {
    fmt.Printf("%+15s: %s\n", k, v)
  }
}
```

output:
```
      timestamp: 23/Apr/2014:22:58:32 +0200
           verb: GET
     rawrequest:
          bytes: 207
           auth: -
        request: /index.php
    httpversion: 1.1
       response: 404
COMMONAPACHELOG: 127.0.0.1 - - [23/Apr/2014:22:58:32 +0200] "GET /index.php HTTP/1.1" 404 207
       clientip: 127.0.0.1
          ident: -
```

# Example 3 - nested
```go
package main

import (
	"fmt"
	"encoding/json"
	"github.com/securitym0nkey/grok"
)

func main() {
	g, _ = grok.NewWithConfig(&grok.Config{NamedCapturesOnly: true})
	nested_values,_ := g.ParseTyped("%{TIME:time_stamp}: %{USER:[name][first_name]} is %{POSINT:[person][age]:int} years old and %{NUMBER:[person][height]:float} meters tall",`12:23:31: bob is 23 years old and 4.2 meters tall`)

	j, _ := json.MarshalIndent(nested_values, "", "\t")
	fmt.Println(string(j))
}
```

output:
```
{
	"name": {
		"first_name": "bob"
	},
	"person": {
		"age": 23,
		"height": 4.2
	},
	"time_stamp": "12:23:31"
}
```
