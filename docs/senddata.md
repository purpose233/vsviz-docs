# Send data

VSVIZ uses TCP sockets to receive raw data from data source. So it won't limit the languages or frameworks for your data source.

## Sending example

You can use any other languages & frameworks to do this job which only contains two things:
 - Wrap your data
 - Send it through TCP socket 

Here is a example of sending video data by Python & OpenCV:

```python
import socket
import time
import cv2 as cv
import numpy as np
import sys
import json

SERVER = ("127.0.0.1", 9000)
TIME_INTERVAL = 30
HEADER_SIZE = 36
PACKAGE_INIT_CODE = 'ADAM'

def copyByte(src, target, targetOffset, srcStart, srcEnd):
  i = srcStart
  j = targetOffset
  maxLen = min(len(target) - targetOffset, srcEnd - srcStart, len(src) - srcStart)
  while i < srcStart + maxLen:
    target[j] = src[i]
    j += 1
    i += 1

def wrapData(info, data, dataType):
  size = len(PACKAGE_INIT_CODE) + HEADER_SIZE + len(data)
  b = bytearray(size)
  copyByte(bytearray(PACKAGE_INIT_CODE, 'utf-8'), b, 0, 0, 4)
  copyByte(bytearray(info['id'], 'utf-8'), b, 4, 0, 8)
  copyByte(bytearray(info['streamType'], 'utf-8'), b, 12, 0, 8)
  copyByte(bytearray(info['dataType'], 'utf-8'), b, 20, 0, 8)
  copyByte(to_bytes(len(data), 4), b, 28, 0, 8)
  copyByte(to_bytes(info['sequence'], 4), b, 32, 0, 8)
  copyByte(to_bytes(info['timestamp'], 4), b, 36, 0, 8)
  if dataType == 'string':
    copyByte(bytearray(data, 'utf-8'), b, 40, 0, len(data))
  else:
    copyByte(bytearray(data), b, 40, 0, len(data))
  return b

def send(s, info, data, dataType):
  s.send(wrapData(info, data, dataType))

if __name__ == "__main__":

  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  s.connect(SERVER)

  if sys.argv[1] == 'meta':
    info = {
      'id': 'video0',
      'streamType': 'meta',
      'dataType': 'json',
      'sequence': 0,
      'timestamp': 0
    }
    dataOrigin = {
      'width': 640,
      'height': 480
    }
    data = json.dumps(dataOrigin)
    send(s, info, data, 'string')
  else:
    cv.namedWindow('camera', 1)
    capture = cv.VideoCapture(0) 
    capture.set(3, 640)
    capture.set(4, 480)

    info = {
      'id': 'video0',
      'streamType': 'video',
      'dataType': 'jpg',
      'sequence': 1,
      'timestamp': 1
    }

    while(True):
      ref, frame = capture.read()
      cv.imshow('camera', frame)

      img = cv.imencode('.jpg', frame)[1]
      data = np.array(img)
      send(s, info, data, 'array')

      info['sequence'] += 1
      info['timestamp'] += 1

      c = cv.waitKey(TIME_INTERVAL)
      if c != -1:
        break

    capture.release()
    s.close()
```

The code above simply get frame data from camera, wrap meta data and video data into binary package and send it through TCP socket.

It works fine with the example shown in [Quick start](quickstart.md). 

## Package Format

The format of binary package could divided into three part: package init code, header, body.

**Package init code**:
The init code of package is `"ADAM"` which occupy 4 bytes. This init code is used to find the start position of package, and it is necessary for data source to wrap data.

**Header**:
The header occupies 36 bytes:

| **entry** | **size** | **format** | **description** |
| - | - | - | - |
| **ID** | 8 Bytes | utf-8 | the id of data stream |
| **stream_type** | 8 Bytes | utf-8 | the type of data stream |
| **data_type** | 8 Bytes | utf-8 | the type of data format |
| **size** | 4 Bytes | uint, big endian | the size of package data |
| **sequence** | 4 Bytes | uint, big endian | the sequence of package |
| **timestamp** | 4 Bytes | uint, big endian | the time of package |

The types of **stream_type**:

| **type name** | **description** |
| - | - |
| meta | Meta data for streams, such as the video size for video stream |
| video | Video stream |
| customed | Customed stream |

The types of **data_type**:

| **type name** | **description** |
| - | - |
| binary | binary data |
| string | string data, encoded by utf-8  |
| json | json data, encoded by utf-8 |
| jpeg | jpeg data of image |
| rgb | rgb data of image |
| bgr | bgr data of image, generally used in opencv |
| rgba | rgba data of image |

**Body**:

Body size is defined by size entry in header, and filled with the package data.
