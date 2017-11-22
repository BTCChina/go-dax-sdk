# go-dax-sdk
dax.btcc.com trading api sdk

### Useage:
first
```
go get -u github.com/JinAirsOs/go-dax-sdk/dax
```

for websocket api you may need
```
go get -u github.com/gorilla/websocket
go get -u github.com/satori/go.uuid
```

here is an example,you may edit it in github.com/JinAirsOs/go-dax-sdk/example/dax.go

```
package main

import (
	"log"
	"os"
	"os/signal"
	"time"

	"github.com/JinAirsOs/go-dax-sdk/dax"
)

const (
	DAX_API_KEY    = ""
	DAX_API_SECRET = ""
)

func main() {
	daxClient := dax.New(DAX_API_KEY, DAX_API_SECRET)

	interrupt := make(chan os.Signal, 1)
	signal.Notify(interrupt, os.Interrupt)

	//get account info
	resp := daxClient.GetAccountInfo()
	log.Println(string(resp.Body))

	//get my open orders
	resp = daxClient.GetMyOrders("ETH_BTC")
	log.Println(string(resp.Body))

	//place an order
	resp = daxClient.PlaceOrder(
		dax.Order{CurrencyPair: "ETH_BTC", // currency pair
			Price:     "0.11111", //order price
			Quantity:  "0.00001", // order quantity
			Type:      "LIMIT",   // "LIMIT"
			BuyOrSell: "SELL",
		}) //buy or sell
	log.Println(string(resp.Body))

	/*{"symbol":"ETH_BTC","price":"0.11111","quantity":"0.00001","type":"LIMIT","side":"SELL"}*/
	//cancel an order
	resp = daxClient.CancelOrder("ETH_BTC", "0123456789abcdef0123456789abcdef")
	log.Println(string(resp.Body))

	//websocket client

	//websocket
	stop := make(chan struct{})
	//defer close(stop)
	dataCh := make(chan []byte, 16)
	go daxClient.SubscribeExchange("ETH_BTC", dataCh, stop)
	go func() {
		for data := range dataCh {
			log.Println(string(data))
		}
	}()
	select {
	case <-interrupt:
		close(stop)
		select {
		case <-time.After(2 * time.Second):
		}
	case <-stop:
		log.Println("dax websocket closed")
		select {
		case <-time.After(2 * time.Second):
		}
	}
}
```

Donate
------
Bitcoin

![Donation QR](http://api.qrserver.com/v1/create-qr-code/?size=200x200&data=bitcoin:1D1m9mRHegmBZnqqUVMZPauqaUqvhQfgod%3Flabel%3DStalker)

[1D1m9mRHegmBZnqqUVMZPauqaUqvhQfgod](http://tinyurl.com/ybeg28qq)