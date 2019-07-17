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

# send canvas data from source
$ python3 ./src/dataCanvas.py data

# send jpeg video data from source (opencv is required)
$ python3 ./src/dataJPEG.py meta
$ python3 ./src/dataJPEG.py data
```
