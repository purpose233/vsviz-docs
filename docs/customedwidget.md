# Customed Widget

@vsviz/ui module provides with several OOTB widget, like video widget and some widgets which features could be expanded.
More details, please check out [@vsviz/ui](ui.md).

You could also create your customed widget which could connect to stream data. 
It is similar to creating react component, here is an example:

```typescript
import { 
  ConnectComponent, 
  BaseWidget, 
  BaseWidgetPropsType
} from '@vsviz/ui';

class MyConnectWidget extends BaseWidget<BaseWidgetPropsType> {

  public onInit() {
    console.log('Widget init.');
  }

  public renderNodes(loaderDataMap: Map<string, LoaderDataType>): React.ReactNode {
    console.log('Widget render. data: ', loaderDataMap);
    return (
      <h1>Hello World!</h1>
    );
  }

  public render(): React.ReactNode {
    return (
      <ConnectComponent 
        loader={this.props.loader}
        dataIds={this.props.dataIds}
        renderNodes={this.renderNodes.bind(this)}
        onInit={this.onInit.bind(this)}
      />
    )
  }
}
```

The difference between vsviz widget and react component is that: 
The `render` function of vsviz widget need to return `ConnectComponent`, and the actually ReactNode is returned by `renderNodes` function. 

The `renderNodes` function will be called each time when stream data reached. 
And the `dataIds` property of `ConnectComponent` is used to filter the streams.
