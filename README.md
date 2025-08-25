 This is a simple tutorial on how to decode the data sent through the Spot WebSocket, I'm new to WebSockets and this might not be the best way to do it, but it works fine for me. This tutorial was made on linux and using Nodejs.

#### What you will need:
- Terminal

  
- The .proto file, you can get it [here](https://github.com/mexcdevelop/websocket-proto). Download the one needed, to find wich one you need click [here](https://mexcdevelop.github.io/apidocs/spot_v3_en/#websocket-market-streams) and look for "Request Parameter", then download the .proto corresponding to the parameter.

  
- protobufjs, you can install it using  <pre>```sudo apt install protobuf-compiler```</pre>


- ts-proto, you can install it using <pre>```npm i ts-proto```</pre>


#### Next step:
- Navigate using the terminal to the folder your .proto file is located, then run this comand to create the decoder file (make sure Nodejs is installed and you are in your project folder):  <pre>```protoc --plugin=../node_modules/.bin/protoc-gen-ts_proto --ts_proto_out=. ./PublicAggreBookTickerV3Api.proto```</pre>

- #### Pay attention to the '../node_modules/' path, change it if needed ( you probably will ).
- #### Also, pay attention to this part '.PublicAggreBookTickerV3Api.proto', each .proto name is differente, change if needed.


- After running the code, a .ts file will appear, you can now delete the .proto file. This new file is the decoder you need.

#### Using the WebSocket 

- This is a simple script connecting to the WebSocket:
  
 ```ts
import WebSocket from "ws"

const url = "wss://wbs-api.mexc.com/ws";
const ws = new WebSocket(url);

 ws.on("open", () => {
    console.log("Connected to Mexc Spot");

    const sub = {
        method: "SUBSCRIPTION",
        params: ["spot@public.aggre.bookTicker.v3.api.pb@100ms@BTCUSDT"],
      };
      ws.send(JSON.stringify(sub));
  });

ws.on("message", (msg: WebSocket.RawData) => {
    
  });
```

### Importing the decode function:

```ts
import { BookTicker } from "../../proto/PublicAggreBookTickerV3Api.ts";
```

- This import is ***important***, because without it you can't decode the data.
  Remember, this is the file we got **after** using protobuf.

#### Decode time

- Now, it's the 'fun' part

  ```ts
   ws.on("message", (msg: WebSocket.RawData) => {
    if (Buffer.isBuffer(msg)) {
      const bufCheck = Buffer.from(msg).toString("utf-8");

      if (!bufCheck.includes("id")) {
        const Uint8 = new Uint8Array(msg);
        const decode = BookTicker.decode(Uint8);

        console.log(decode);
      }
    }
  });
  ```
- Here's the explanation:

  Firstly, we need to check if the message is a Buffer, because we can't use ```Buffer.from``` on a RawData type, this is for the TypeScript to shut up.

  
  Secondly, the first message that come from the WebSocket is a 'success message', not the data we want, so I added a bufCheck and checked if it has an 'id' inside it, if it does then it is the 'success' warning, if not then it is the stuff we want.


  After the check, we transform the message into a ```Uint8Array```, we need to do this so we can decode the message.


  And for the decoding part, we use the 'BookTicker.decode' function to decode the ```Uint8Array```.


  The log should be something like this:

  ```bash
  
  {
  channel: 'spot@public.aggre.bookTicker.v3.api.pb@100ms@BTCUSDT',
  publicbookticker: {
    bidPrice: '0.0303',
    bidQuantity: '114.38',
    askPrice: '0.03042',
    askQuantity: '65.17'
  },
  symbol: 'BTCUSDT',
  sendtime: 1756105282327
  ```


  #### And thats it :D

  - Just remember that you are supposed to send a 'ping' message every ~15 seconds to not get disconnected!
