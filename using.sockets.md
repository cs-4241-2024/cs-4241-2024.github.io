# Socket Servers w/ Svelte (aka realtime chat)

## What are Web Sockets?
- TCP and require a handshake to establish connection
- realtime communication tech (chat, multiplayer games etc.)
- remember to implement heartbeats to maintain connections
- [socket.io](https://socket.io/) is a great library that abstracts sockets across programming languages, but they're easy to use in JS without additional libraries.

## A quick server for both http and WebSockets (ws)

Two options for dealing with ws:
[ws](https://www.npmjs.com/package/ws) - A node package for creating servers that understand the WebSocket protocol
[express-ws](https://www.npmjs.com/package/express-ws) - A node package for handling WebSocket routes using standard
Express APIs. 

We'll use `ws`... just because that's what I'm more familiar with. `express-ws` looks like a great option though!

To setup the project:
1. Run `npm init vite` and make a new Svelte + JavaScript project.
2. `cd` into the new directory, and run `npm install` to get all the Vite dependencies
3. Get our express / websocket dependencies: `npm install express vite-express ws`

In order to get our setup to work with the Vite dev server, we have to `bind` ViteExpress to our http server instead
of having ViteExpress do the listening itself. Create a `server.js` file in the top level of your project directory and paste
the following server code inside:

```js
/* 
1. Open up a socket server
2. Maintain a list of clients connected to the socket server
3. When a client sends a message to the socket server, forward it to all
connected clients
*/

import express from 'express'
import http from 'http'
import ViteExpress from 'vite-express'
import { WebSocketServer } from 'ws'

const app = express()

const server = http.createServer( app ),
      socketServer = new WebSocketServer({ server }),
      clients = []

socketServer.on( 'connection', client => {
  console.log( 'connect!' )
    
  // when the server receives a message from this client...
  client.on( 'message', msg => {
	  // send msg to every client EXCEPT the one who originally sent it
    clients.forEach( c => { if( c !== client ) c.send( msg ) })
  })

  // add client to client list
  clients.push( client )
})

server.listen( 3000 )

ViteExpress.bind( app, socketServer )
```

## Svelte Client
Go ahead and start your new server with `node server.js`; this will also startup the Vite development server. With 
the Vite development server running, replace all the code in your App.svelte component with the following:

```js
<script>
  let msgs = []
      
  const ws = new WebSocket( 'ws://127.0.0.1:3000' )

  // when connection is established...
  ws.onopen = () => {
    ws.send( 'a new client has connected.' )

    ws.onmessage = async msg => {
      // add message to end of msgs array,
      // re-assign to trigger UI update
      const message = await msg.data.text()
      msgs = msgs.concat([ 'them: ' + message ])
    }
  }

  const send = function() {
    const txt = document.querySelector('input').value
    ws.send( txt )
    msgs = msgs.concat([ 'me: ' + txt ])
  }
</script>

<input type='text' on:change={send} />

{#each msgs as msg }
  <h3>{msg}</h3>
{/each}
```

If you open the Svelte app in two different tabs / browsers you'll be able to see that you can send messages back and forth. Realtime
communication is ready to go.

There are lots of applications where a simple relay server can be useful with WebSocket messages. For example, we can use the exact same 
server to make a simple “drawing” application. We’ll create a `<canvas>` object that fills the entire window, and then send the current 
mouse position whenever the window is clicked. We’ll then add code to take any message received and draw a box at the provided location.

```html
<html lang="en">
  <head>
    <style>
      body { 
        margin:0; 
        background:black 
      } 
    </style>
    <script>
      let ws, msgs = [], ctx = null
      
      window.onload = function() {
        ws = new WebSocket( 'ws://127.0.0.1:3000' )

        ws.onopen = () => {
          ws.onmessage = async msg => {
            const pos = await msg.data.text()
            const [x,y] = pos.split( ':' ).map( v => parseInt(v) )

            ctx.fillStyle = 'red'
            ctx.fillRect( x,y,50,50 )
          }
        }

        const canvas = document.querySelector('canvas')
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
        ctx = canvas.getContext( '2d' )

        window.onclick = e => {
          ws.send( `${e.pageX}:${e.pageY}` )
          ctx.fillStyle = 'yellow'
          ctx.fillRect( e.pageX,e.pageY,50,50 )
        }
      }
    </script>
  </head>
  <body>
    <canvas></canvas>
  </body>
</html>
```
