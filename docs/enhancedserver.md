# Enhanced Server

[Quick start](quickstart.md) has shown a simple server which will forward data through TimerSender middleware.

?> Check out [@vsviz/server](server.md) for more built-in middlewares.

@vsviz/server module provides with several built-in middlewares, which could be enough for simple applications.

You can also create your own middleware to enhance the features server. And all middlewares are **event-driven**.

## VizServer

You can create server instance by `VizServer` class. It handles all sockets which come from data sources and all sessions with web clients. 

The event types of middleware are:
 - **timeout**: called when reached interval time which is set by developer, the function to override is `protected async onTimeout(next: Function, context: MiddlewareContext): Promise<void>`.
 - **streamMsg**: called when the server has received one or several packages from data source, the function to override is `protected async onStreamMessage(next: Function, context: MiddlewareContext, msg: StreamMessageType[]): Promise<void>`;
 - **connection**: called on session connection reached, the function to override is `protected async onConnection(next: Function, context: MiddlewareContext, msg: IncomingMessage): Promise<void>`;
 - **close**: called on session closed, the function to override is `protected async onClose(next: Function, context: MiddlewareContext): Promise<void>`;
 - **sessionMsg**: called on received message from web client, the function to override is `protected async onSessionMessage(next: Function, context: MiddlewareContext, msg: WebSocket.Data | ClientMessageType): Promise<void>`.

## Customed Middleware

You can create your own middleware for data server or ws server. All middleware use `async/await` to control the asynchronous process and decide whether to call next middleware.

Two types of middlewares are suppored: **function middleware** and **class middleware**.

Here is a simple customed function middleware:

```typescript
import { MiddlewareContext } from '@vsviz/server';

async function MyFunctionMiddleware(next: Function, type: TimerEventEnum, context: MiddlewareContext, msg: any): Promise<void> {
  if (type === 'connection') {
    console.log('Session has created.');
  }
  await next();
}
``` 

The function middleware need to handle all events, the arguments are: `next: Function, type: MiddlewareEventType, context: MiddlewareContext, msg: any`.

The class middleware need to extends parent class `BaseMiddleware`.

Here is a simple customed class middleware: 

```typescript
import { BaseMiddleware, MiddlewareContext } from '@vsviz/server';

class MyMiddleware extends BaseMiddleware {

  private lastTime: number = 0;

  protected async onStreamMessage(next: Function, context: MiddlewareContext, data: any): Promise<void> {
    console.log('received data. ');
    if (this.lastTime === 0) {
      this.lastTime = new Date().getTime();
    } else {
      const currentTime = new Date().getTime();
      console.log('Time interval: ', currentTime - this.lastTime);
      this.lastTime = currentTime;
    }
    console.log('package size: ' + (data[0].info.size / 1024).toFixed(1) + 'kb');
    await next();
  }
}
```

You can override one or several event callbacks, and the rest will do nothing but pass to the next middleware.

!> **DO NOT FORGET** to use `await next()` to call next middleware unless you are quite sure about what is going to happen.

!> **DO NOT** modify the arguments passed to your customed middleware callbacks which will influence all the next middlewares.

### Built-in Middleware Context Data

The argument context for all middlewares is used for store data which could be fetch by all middlewares. It is a key-value map which provides `get` and `set` function. 

Here are some built-in context data for all middlewares:
 - Builder: the builder which could store parsed data and wrap them to binary package, the key is `Symbol.for('builder')`.
 - Socket: the web socket connection from client, the key is `Symbol.for('socket')`;
 - SessionID: the session id for session, the key is `Symbol.for('SessionId')`.

### Arguments of Middleware Context

You can simply pass the class name of middleware to `use` function to enable it. 

But when your customed middleware class contructor need to take some arguments, you will have to override the `copy` function of base middleware class.

Here is en example:

```typescript
import { BaseMiddleware } from '@vsviz/server';

class MyMiddleware extends BaseMiddleware {
  private name: string;

  contructor(str: string) {
    this.name = str;
  }

  public copy(): MyMiddleware {
    return new MyMiddleware(this.name);
  }
}
```

### Singleton Middleware

Sometimes you need a middleware to process stream data only once rather than processing multiple times in independent sessions.

Here we provide with singleton middleware, it only responds to **streamMsg** and **timeout** event, and the instance won't be created multiple times.

And it's quite handful to use, you only need to set the second parameter of `use` function to true, like this: `Server.use(SingletonMiddleware, true)`.
