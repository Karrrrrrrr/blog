---
title: "kataras/neffos"

date: "2023-01-19T18:41:33+08:00"

draft: false

---

用来替代socketio的东西, 因为socketio没有go的最新实现

实例代码如下, 注意以下代码必须同域使用, 跨域则无法连接


<a src="https://github.com/kataras/neffos" > 服务器端源码</a>

```go
package main

import (
	"bufio"
	"bytes"
	"context"
	"fmt"
	"log"
	"net/http"
	"os"
	"time"

	"github.com/kataras/neffos"
	"github.com/kataras/neffos/gobwas"
	"github.com/kataras/neffos/gorilla"
)

/*
	$ go run main.go server
	# new tab(s) and:
	$ go run main.go client
*/

const (
	endpoint  = "localhost:9090"
	namespace = "default"
	timeout   = 0 // 30 * time.Second
)

var handler = neffos.WithTimeout{
	ReadTimeout:  timeout,
	WriteTimeout: timeout,
	Namespaces: neffos.Namespaces{
		"default": neffos.Events{
			neffos.OnNamespaceConnect: func(c *neffos.NSConn, msg neffos.Message) error {
				if msg.Err != nil {
					log.Printf("This client can't connect because of: %v", msg.Err)
					return nil
				}

				err := fmt.Errorf("Server says that you are not allowed here")
				/* comment this to see that the server-side will
				no allow to for this socket to be connected to the "default" namespace
				and an error will be logged to the client. */
				err = nil

				return err
			},
			neffos.OnNamespaceConnected: func(c *neffos.NSConn, msg neffos.Message) error {
				if !c.Conn.IsClient() {
					c.Emit("chat", []byte("welcome to server's namespace"))
				}

				log.Printf("[%s] connected to [%s].", c.Conn.ID(), msg.Namespace)

				return nil
			},
			neffos.OnNamespaceDisconnect: func(c *neffos.NSConn, msg neffos.Message) error {
				if msg.Err != nil {
					log.Printf("This client can't disconnect yet, server does not allow that action, reason: %v", msg.Err)
					return nil
				}

				err := fmt.Errorf("Server says that you are not allowed to be disconnected yet")
				/* here if you comment this, the return error will mean that
				the disconnect message from client-side will be ignored from the server
				and the connection would be still available to send message to the "default" namespace
				it will not be disconnected.*/
				err = nil

				if err == nil {
					log.Printf("[%s] disconnected from [%s].", c.Conn.ID(), msg.Namespace)
				}

				if c.Conn.IsClient() {
					os.Exit(0)
				}

				return err
			},
			"chat": func(c *neffos.NSConn, msg neffos.Message) error {
				if !c.Conn.IsClient() {
					// this is possible too:
					// if bytes.Equal(msg.Body, []byte("force disconnect")) {
					// 	println("force disconnect")
					// 	return c.Disconnect()
					// }

					log.Printf("--server-side-- send back the message [%s:%s]", msg.Event, string(msg.Body))
					//	c.Emit(msg.Event, msg.Body)
					//	c.Server().Broadcast(nil, msg) // to all including this connection.
					c.Conn.Server().Broadcast(c.Conn, msg) // to all except this connection.
				}

				log.Printf("---------------------\n[%s] %s", c.Conn.ID(), msg.Body)
				return nil
			},
		},
	},
}

func main() {
	args := os.Args[1:]
	if len(args) == 0 {
		log.Fatalf("expected program to start with 'server' or 'client' argument")
	}
	side := args[0]

	var (
		upgrader = gobwas.DefaultUpgrader
		dialer   = gobwas.DefaultDialer
	)

	if len(args) > 1 {
		method := args[1]
		if method == "gorilla" {
			upgrader = gorilla.DefaultUpgrader
			dialer = gorilla.DefaultDialer
			if side == "server" {
				log.Printf("Using with Gorilla Upgrader.")
			} else {
				log.Printf("Using with Gorilla Dialer.")
			}
		}
	}

	switch side {
	case "server":
		server(upgrader)
	case "client":
		client(dialer)
	default:
		log.Fatalf("unexpected argument, expected 'server' or 'client' but got '%s'", side)
	}
}

var (
	// tests immediately closed on the `Server#OnConnect`.
	dissalowAll = false
	// if not empty, tests broadcast on `Server#OnConnect` (expect this conn because it is not yet connected to any namespace locally).
	notifyOthers                  = true
	serverHandlesConnectNamespace = true
)

func server(upgrader neffos.Upgrader) {
	srv := neffos.New(upgrader, handler)
	// s, err := redis.NewStackExchange(redis.Config{}, "ch")
	// if err != nil {
	// 	log.Fatal(err)
	// }
	// srv.UseStackExchange(s)

	srv.OnConnect = func(c *neffos.Conn) error {
		if dissalowAll {
			return fmt.Errorf("you are not allowed to connect here for some reason")
		}

		log.Printf("[%s] connected to server.", c.ID())

		if serverHandlesConnectNamespace {
			ns, err := c.Connect(nil, namespace)
			if err != nil {
				panic(err)
			}

			ns.Emit("chat", []byte("(Force-connected by server)"))
		}

		if notifyOthers {
			c.Server().Broadcast(c, neffos.Message{
				Namespace: namespace,
				Event:     "chat",
				Body:      []byte(fmt.Sprintf("Client [%s] connected too.", c.ID())),
			})

			// c.Server().Broadcast(c, neffos.Message{
			// 	Namespace: namespace,
			// 	Event:     "chat",
			// 	Body:      []byte(fmt.Sprintf("SECOND ONE")),
			// })
		}

		return nil
	}

	srv.OnDisconnect = func(c *neffos.Conn) {
		log.Printf("[%s] disconnected from the server.", c.ID())
	}

	srv.OnUpgradeError = func(err error) {
		log.Printf("ERROR: %v", err)
	}

	log.Printf("Listening on: %s\nPress CTRL/CMD+C to interrupt.", endpoint)
	go http.ListenAndServe(endpoint, srv)

	fmt.Fprint(os.Stdout, ">> ")
	scanner := bufio.NewScanner(os.Stdin)
	for {
		if !scanner.Scan() {
			log.Printf("ERROR: %v", scanner.Err())
			return
		}

		text := scanner.Bytes()
		if bytes.Equal(text, []byte("force disconnect")) {
			srv.Do(func(c *neffos.Conn) {
				c.DisconnectAll(nil)
				//	c.Namespace(namespace).Disconnect(nil)
			}, false)
		} else {
			// srv.Do(func(c neffos.Conn) {
			// 	c.Write(namespace, "chat", text)
			// }, false)
			srv.Broadcast(nil, neffos.Message{Namespace: namespace, Event: "chat", Body: text})
		}
		fmt.Fprint(os.Stdout, ">> ")
	}
}

const dialAndConnectTimeout = 5 * time.Second

func client(dialer neffos.Dialer) {
	ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(dialAndConnectTimeout))
	defer cancel()

	client, err := neffos.Dial(ctx, dialer, endpoint, handler)
	if err != nil {
		panic(err)
	}

	defer client.Close()

	var c *neffos.NSConn

	if serverHandlesConnectNamespace {
		c, err = client.WaitServerConnect(ctx, namespace)
	} else {
		c, err = client.Connect(ctx, namespace)
	}

	if err != nil {
		panic(err)
	}

	fmt.Fprint(os.Stdout, ">> ")
	scanner := bufio.NewScanner(os.Stdin)
	for {
		if !scanner.Scan() {
			log.Printf("ERROR: %v", scanner.Err())
			return
		}

		text := scanner.Bytes()

		if bytes.Equal(text, []byte("exit")) {
			if err := c.Disconnect(nil); err != nil {
				// log.Printf("from server: %v", err)
			}
			continue
		}

		ok := c.Emit("chat", text)
		if !ok {
			break
		}

		fmt.Fprint(os.Stdout, ">> ")
	}
}

```

<img src="https://camo.githubusercontent.com/dda875c9cddcbabdbba24ab97cbf6be158c7e2a93c462402f279e55c7de9ea83/68747470733a2f2f697269732d676f2e636f6d2f696d616765732f6e6566666f732d626f6f6b2d6f766572766965772e706e67"/>

---

<a src="https://github.com/kataras/neffos.js" > 前端源码 </a>

```shell
pnpm add @types/neffos.js neffos.js
```

```html
<!-- the message's input -->
<input id="input" type="text" />

<!-- when clicked then a websocket event will be sent to the server, at this example we registered the 'chat' -->
<button id="sendBtn" disabled>Send</button>

<!-- the messages will be shown here -->
<pre id="output"></pre>
<!-- import the iris client-side library for browser from a CDN or locally.
     However, `neffos.(min.)js` is a NPM package too so alternatively,
     you can use it as dependency on your package.json and all nodejs-npm tooling become available:
     see the "browserify" example for more-->
<script src="https://cdn.jsdelivr.net/npm/neffos.js@latest/dist/neffos.min.js"></script>
<script>
    // `neffos` global variable is available now.

    var scheme = document.location.protocol == "https:" ? "wss" : "ws";
    var port = document.location.port ? ":" + document.location.port : "";
    var wsURL = scheme + "://" + document.location.hostname + port + "/echo";

    var outputTxt = document.getElementById("output");
    function addMessage(msg) {
        outputTxt.innerHTML += msg + "\n";
    }

    function handleError(reason) {
        console.log(reason);
        window.alert(reason);
    }

    function handleNamespaceConnectedConn(nsConn) {
        let inputTxt = document.getElementById("input");
        let sendBtn = document.getElementById("sendBtn");

        sendBtn.disabled = false;
        sendBtn.onclick = function () {
            const input = inputTxt.value;
            inputTxt.value = "";
            nsConn.emit("chat", input);
            addMessage("Me: " + input);
        };
    }

    async function runExample() {
        var events = new Object();
        events._OnNamespaceConnected = function (nsConn, msg) {
            if (nsConn.conn.wasReconnected()) {
                addMessage("re-connected after " + nsConn.conn.reconnectTries.toString() + " trie(s)");
            }

            addMessage("connected to namespace: " + msg.Namespace);
            handleNamespaceConnectedConn(nsConn);
        }

        events._OnNamespaceDisconnect = function (nsConn, msg) {
            addMessage("disconnected from namespace: " + msg.Namespace);
        }

        events.chat = function (nsConn, msg) { // "chat" event.
            addMessage(msg.Body);
        }

        /* OR regiter those events as:
            neffos.dial(wsURL, {default: {
                chat: function (nsConn, msg) { [...] }
            }});
        */

        try {
            // You can omit the "default" namespace and simply define only Events,
            // the namespace will be an empty string"",
            // however if you decide to make any changes on
            // this example make sure the changes are reflecting inside the ../server.go file as well.
            //
            // At "wsURL" you can put the relative URL if the client and server
            // hosted in the same address, e.g. "/echo".
            const conn = await neffos.dial(wsURL, { default: events }, {
                // if > 0 then on network failures it tries to reconnect every 5 seconds, defaults to 0 (disabled).
                reconnect: 5000,
                // custom headers:
                // headers: {
                //    'X-Username': 'kataras',
                // }
            });

            // You can either wait to conenct or just conn.connect("connect")
            // and put the `handleNamespaceConnectedConn` inside `_OnNamespaceConnected` callback instead.
            // const nsConn = await conn.connect("default");
            // handleNamespaceConnectedConn(nsConn);
            conn.connect("default");

        } catch (err) {
            handleError(err);
        }
    }

    runExample();

    // If "await" and "async" are available, use them instead^, all modern browsers support those,
    // so all of the examples will be written using async/await method instead of promise then/catch callbacks.
    // A usage example of promise then/catch follows:
    // neffos.dial(wsURL, {
    //     default: { // "default" namespace.
    //         _OnNamespaceConnected: function (ns, msg) {
    //             addMessage("connected to namespace: " + msg.Namespace);
    //         },
    //         _OnNamespaceDisconnect: function (ns, msg) {
    //             addMessage("disconnected from namespace: " + msg.Namespace);
    //         },
    //         chat: function (ns, msg) { // "chat" event.
    //             addMessage(msg.Body);
    //         }
    //     }
    // }).then(function (conn) {
    //     conn.connect("default").then(handleNamespaceConnectedConn).catch(handleError);
    // }).catch(handleError);
</script>
```