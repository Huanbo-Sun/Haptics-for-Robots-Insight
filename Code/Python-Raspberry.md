```
# Packages
## Neopixel
import board
from fractions import Fraction
import neopixel
## Picamera
import picamera
from picamera import PiCamera, Color
from picamera.array import PiRGBArray
## Communication
import socket
import io
import struct
import pickle
## Processing
import numpy as np
import math
import time

# Camera
def imaging():
    camera = PiCamera(resolution=(1640,1232),framerate=40)
    camera.iso = 100
    time.sleep(2)
    camera.shutter_speed = camera.exposure_speed
    camera.exposure_mode = 'off'
    camera.awb_mode = 'off'
    camera.awb_gains = (Fraction(387,256), Fraction(477,256))
    return camera

# Light
def lightning():
    pixels_M = neopixel.NeoPixel(board.D18,8,auto_write=True)
    pr = 1.0#1.0
    pg = 1.0#0.9
    pb = 0.5
    b0 = int(255.0*pr)
    b1 = int(255.0*pg)
    b2 = int(255.0*pb)
    b3 = int(255.0*pr)
    b4 = int(255.0*pg)
    b5 = int(255.0*pr)
    b6 = int(255.0*pb)
    b7 = int(255.0*pg)
    pixels_M[0]=(b0,0,0)
    pixels_M[1]=(0,b1,0)
    pixels_M[2]=(0,0,b2)
    pixels_M[3]=(b3,0,0)
    pixels_M[4]=(0,b4,0)
    pixels_M[5]=(b5,0,0)
    pixels_M[6]=(0,0,b6)
    pixels_M[7]=(0,b7,0)
    pixels_M.write()
    time.sleep(0.025)

# Main
client_socket = socket.socket()
client_socket.connect(('--.--.--.--', ----))
a = client_socket.recv(256)
print(a)
connection = client_socket.makefile('wb')

lightning()
camera = imaging()
camera.start_preview()
time.sleep(0.1)

try:
    while True:
        a = client_socket.recv(256)
        stream = io.BytesIO()
        camera.capture(stream,'jpeg',use_video_port=True)
        connection.write(struct.pack('<L',stream.tell()))
        connection.flush()
        stream.seek(0)
        connection.write(stream.read())
        stream.seek(0)
        stream.truncate()
finally:
    time.sleep(1)
    connection.close()
    client_socket.close()
```

