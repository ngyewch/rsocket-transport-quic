# rsocket-transport-quic

QUIC transport for rsocket-go.

## Quick Start

> Start Echo Server

```go
package main

import (
	"context"
	"log"

	rtq "github.com/ngyewch/rsocket-transport-quic"
	"github.com/rsocket/rsocket-go"
	"github.com/rsocket/rsocket-go/payload"
	"github.com/rsocket/rsocket-go/rx/mono"
)

func main() {
	err := rsocket.Receive().
		Acceptor(func(ctx context.Context, setup payload.SetupPayload, sendingSocket rsocket.CloseableRSocket) (responder rsocket.RSocket, err error) {
			responder = rsocket.NewAbstractSocket(
				rsocket.RequestResponse(func(request payload.Payload) mono.Mono {
					return mono.Just(request)
				}),
			)
			return
		}).
		Transport(rtq.Server().SetAddr(":12345").Build()).
		Serve(context.Background())
	log.Fatalln(err)
}
```

> Client

```go
package main

import (
	"context"
	"log"

	rtq "github.com/ngyewch/rsocket-transport-quic"
	"github.com/rsocket/rsocket-go"
	"github.com/rsocket/rsocket-go/payload"
)

func main() {
	client, err := rsocket.Connect().
		Transport(rtq.Client().SetAddr("127.0.0.1:12345").Build()).
		Start(context.Background())

	if err != nil {
		panic(err)
	}
	defer client.Close()

	res, err := client.RequestResponse(payload.NewString("hello world", "rsocket")).Block(context.Background())
	if err != nil {
		panic(err)
	}
	log.Println("response:", res.DataUTF8())
}
```
