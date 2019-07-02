# Enhanced Server

[Quick start](quickstart.md) has shown a simple server which will forward data through TimerSender middleware.

?> Check out [@vsviz/server](server.md) for more built-in middlewares.

@vsviz/server module provides with several built-in middlewares, which could be enough for simple applications.

You can also create your own middleware to enhance the features of data server and web socket server. And all middlewares are **event-driven**.

## Data Server

You can create data server by `DataServer` class. It handles all sockets which come from data sources. 

The event types of data server are:
 - **init**: called when the data server is started, the function to override is `protected async onInitial(next: Function, context: MiddlewareContext): Promise<void>`;
 - **end**: called when the data server is closed, the function to override is `protected async onEnd(next: Function, context: MiddlewareContext): Promise<void>`;
 - **data**: called when the data server has received one or several packages, the function to override is `protected async onData(next: Function, msg: ParsedDataType[], context: MiddlewareContext): Promise<void>`;
 - **timeout**: called when reached interval time which is set by developer, the function to override is `protected async onTimeout(next: Function, context: MiddlewareContext): Promise<void>`.

!> Note that sending all data to clients when `data` event is called might be useful enough. But sometimes the data source might send too fast that the clients won't be able to handle. In this case, you can limit the forwarding rate by setting interval time.

## Web Socket Server

You can create web socket server by `WSServer` class. It handles all web socket connected from web clients.

The event types of web socket server are: 
 - **connection**: called on ws connection reached, the function to override is `protected async onConnection(next: Function, msg: IncomingMessage, context: MiddlewareContext): Promise<void>`;
 - **close**: called on ws close the function to override is `protected async onClose(next: Function, context: MiddlewareContext): Promise<void>`;
 - **message**: called on ws send message to server, the function to override is `protected async onMessage(next: Function, msg: WebSocket.Data, context: MiddlewareContext): Promise<void>`.

## Customed Middleware

You can create your own middleware for data server or ws server. All middleware use `async/await` to control the asynchronous process and decide whether to call next middleware.

The middleware for ws server need to inherit the `SessionMiddleware` class. And the middleware for data server need to inherit the `TimerMiddleware` class.

!> Note that the reason why the middleware of data server is named `TimerMiddleware` is that data server provides a clock and will call the callbacks of middlewares when timeout. And this terrible name might be changed in the future.

Here is a simple customed middleware for data server: 

```typescript
class MyTimerMiddleware extends TimerMiddleware {
  private lastTime = 0;

  async onData(next: Function, data: any, context: MiddlewareContext): Promise<void> {
    console.log('received data.');
    if (this.lastTime === 0) {
      this.lastTime = new Date().getTime();
    } else {
      const currentTime = new Date().getTime();
      console.log('Time interval: ', currentTime - this.lastTime);
      this.lastTime = currentTime;
    }

    await next();
  }
}
```

You can override one or several event callbacks, and the rest will do nothing but pass to the next middleware.

!> **DO NOT FORGET** to use `await next()` to call next middleware unless you are quite sure about what is going to happen.

!> **DO NOT** modify the arguments passed to your customed middleware callbacks which will influence all the next middlewares.

### Built-in Middleware Context Data

The argument context for all middlewares is used for store data which could be fetch by all middlewares. It is a key-value map which provides `get` and `set` function. 

<!-- Here are some built-in context data for all middlewares of data server:
 - 

Here are some built-in context data for all middlewares of ws server:
 - key: -->
