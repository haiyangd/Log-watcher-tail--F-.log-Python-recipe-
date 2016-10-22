# Log-watcher-tail--F-.log-Python-recipe-
Basic usage

same as: tail -F /var/log/*.log

def callback(filename, lines):
    for line in lines:
        print(line)

watcher = LogWatcher("/var/log/", callback)
watcher.loop()
Also read last N lines from files before start watching

same as: tail -F /var/log/*.log -n 20

def callback(filename, lines):
    for line in lines:
        print(line)

watcher = LogWatcher("/var/log/", callback, tail_lines=20)
watcher.loop()
Tail last N lines from a single file only

same as: tail -n 10 foo.log

LogWatcher.tail('foo.log', 10)
Non blocking

import time

def callback(filename, lines):
    for line in lines:
        print(line)

watcher = LogWatcher("/var/log/", callback)
while 1:
    print("loop")
    watcher.loop(blocking=False)
    time.sleep(0.1)
Coloured logs

In case your python application is using the logging module you might want to monitor what it's doing in real time and have a coloured ouput. Assuming your log format is configured as such:

 import logging
 logging.basicConfig(level=logging.DEBUG,
                     format='[%(levelname)1.1s %(asctime)s] %(message)s',)
...you'll have log lines looking like this:

[I 2011-11-29 19:26:44,774] info message
[D 2011-11-29 19:26:44,774] debug message
[E 2011-11-29 19:26:44,774] some error message
The code below is able to parse this syntax and add shell colors, including unhandled exception tracebacks which aren't logged via logging.error():

RED = "31m"
BLUE = "34m"
GREEN = "32m"
YELLOW = "33m"
MAGENTA = "35m"

def coloured(s, color):
    return '\033[1;%s%s\033[1;m' % (color, s)

def callback(filename, lines):
    while lines:
        line = lines.pop(0).rstrip()
        noheader = False
        if line.startswith("[E ") or line.startswith("Traceback"):
            color = RED
        elif line.startswith("[D "):
            color = BLUE
        elif line.startswith("[I "):
            color = GREEN
        elif line.startswith("[W "):
            color = YELLOW
        else:
            noheader = True
            color = MAGENTA

        if noheader:
            print(line)
        else:
            endheader = line.find(']')
            header = coloured(line[0:endheader + 1], color)
            line = line[endheader + 1:]
            print(header + line)

watcher = LogWatcher("/var/log/", callback, tail_lines=10)
watcher.loop()
