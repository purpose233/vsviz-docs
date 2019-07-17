# Quick Start

To use the basic features of vsviz, you need to install server & ui modules.

```bash
$ npm install @vsviz/server @vsviz/ui -s
```

# Server

VSVIZ provides with basic data server and web socket server which can be directly used through very few lines.

```typescript
import { VizServer } from '@vsviz/server'

const WSPort: number = 3000;
const DataPort: number = 9000;

const Server = new VizServer({
  sessionPort: WSPort,
  streamPort: DataPort
});
Server.start();
```

Use tsc to compile the code above, and run it by node. And you get your server running, and it will forward data to clients. 

!> Note that though TypeScript is recommended, you can also use JavaScript and replace the `import` syntax by `require`.

## Client

@vsviz/ui provides with several OOTB react components which you can simply place in your application. And of cource, you need to install react.

```typescript  
import React from 'react';
import { render } from 'react-dom'
import { WSLoader, Video } from '@vsviz/ui';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.loader = new WSLoader('ws://localhost:3000/');
  }

  render () {
    return (
      <div>
        <Video 
          loader={this.loader}
          dataIds={['video0']}
          width={640}
          height={480}
        />
      </div>
    );
  }
}

render(<App />, document.getElementById('container'));
```

Use webpack to pack the code above and you get a simple application which can display video data from your data source.

!> For now, you have already created a application with complete FE & BE. But you still need sender to get raw video data.
