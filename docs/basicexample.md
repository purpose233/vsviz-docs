# Basic Example

?> The source is placed on [github](https://github.com/purpose233/vsviz/tree/master/example/basic).

Basic example shows how to collect video data from camera and display it on client.

```bash
$ cd example/basic

# build
$ npm install
$ npm run buildServer
$ npm run buildClient

# run server, it will listen on port 8080
$ npm run runServer

# send data from source
$ python3 ./src/dataSrc00.py meta
$ python3 ./src/dataSrc00.py data
```
