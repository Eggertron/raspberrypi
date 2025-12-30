# OLED 0.91 inch

## Setup
follow the instructions for lgpio here https://www.waveshare.com/wiki/0.91inch_OLED_Module

## Example Code
```python
#!/usr/bin/python
# -*- coding:utf-8 -*-

import sys
import os
picdir = os.path.join(os.path.dirname(os.path.dirname(os.path.realpath(__file__))), 'pic')
libdir = os.path.join(os.path.dirname(os.path.dirname(os.path.realpath(__file__))), 'lib')
if os.path.exists(libdir):
    sys.path.append(libdir)

import logging
import time
import traceback
from waveshare_OLED import OLED_0in91
from PIL import Image,ImageDraw,ImageFont
logging.basicConfig(level=logging.DEBUG)

def init():
    disp = OLED_0in91.OLED_0in91()
    logging.info("\r0.91inch OLED Module ")
    # Initialize library.
    disp.Init()
    # Clear display.
    logging.info("clear display")
    disp.clear()
    return disp

def process(disp) -> None:
    x = 0
    y = 0
    img = Image.new('1', (disp.width, disp.height), "WHITE")
    draw = ImageDraw.Draw(img)
    text = "Hello World!"
    font1 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 12)
    while True:
        x = (x + 1)%disp.width
        #disp.clear()
        draw.rectangle([(0,0), (disp.width, disp.height)],fill="WHITE")
        draw.text((x, y), text, font = font1, fill = 0)
        disp.ShowImage(disp.getbuffer(img))
        time.sleep(1)

def teardown(disp) -> None:
    logging.info("Teardown.")
    disp.clear()
    disp.module_exit()
    exit()

def main() -> None:
    try:
        d = init()
        process(d)
    except IOError as e:
        logging.info(e)
    except KeyboardInterrupt:
        logging.info("ctrl + c:")
    teardown(d)

if __name__ == '__main__':
    main()
```
