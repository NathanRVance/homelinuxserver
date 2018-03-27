---
title: "Youtube Without Youtube"
desc: "A text based youtube like experience"
sortable: 2
---

I have an old desktop. One of the first consumer runs of 64 bit AMD processors, 500 MB RAM, no graphics card. It can't load Firefox without swapping excessively and grinding to a halt. But I don't need it to run Firefox for most things; web browsing works just fine with elinks. What I need it to do is play videos, and videos are on youtube, and youtube needs a cushy web browser like Firefox. Usually.

There are several youtube clients available, any of which could alleviate my troubles. I chose the fun route: youtube-dl piped into mplayer, frontend written in Python. Here's the code.
```
#!/usr/bin/env python3
import argparse
import subprocess
from pathlib import Path
from os import path
import re
from html.parser import HTMLParser

CONF= path.expanduser('~/.utube')

parser = argparse.ArgumentParser(description='Play YouTube vids made by your favourite producers!')
parser.add_argument('-a', '--add', metavar='UTUBR', nargs=1, help='Add a youtuber to those tracked by utube')
parser.add_argument('-u', '--update', action='store_true', help='Update listing of videos')
parser.add_argument('-l', '--list', action='store_true', help='List videos available')
parser.add_argument('-w', '--watch', nargs=1, type=int, help='Watch a specific video')
parser.add_argument('--markAllWatched', action='store_true', help='Mark all videos as watched')
parser.add_argument('--unwatch', nargs=1, type=int, help='Mark a specific video as unwatched')
args = parser.parse_args()

def readConf():
    utubrs = {} # { utubr : [ [name, duration, hasWatched], ...] }
    # Ensure it exists!
    Path(CONF).touch()
    with open(CONF) as f:
        utubr = ''
        for line in f.read().split('\n'):
            if not utubr and line:
                utubr = line
                utubrs[utubr] = []
            elif not line:
                utubr = ''
            else:
                (vid, line, watched) = line.rsplit('&', 2)
                if watched == 'True':
                    watched = True
                else:
                    watched = False
                utubrs[utubr].append([vid, line, watched])
    return utubrs

def writeConf(utubrs):
    with open(CONF, 'w') as f:
        for utubr in sorted(utubrs):
            f.write(utubr + '\n')
            for vid in utubrs[utubr]:
                f.write('{}&{}&{}\n'.format(vid[0], vid[1], vid[2]))
            f.write('\n') # blank line

def getVid(index):
    i = 0
    for utubr in sorted(utubrs):
        for vid in utubrs[utubr]:
            if i == index:
                return vid
            i += 1

def printFancy(vids = [], printAll = True):
    index = 0
    for utubr in sorted(utubrs):
        print('\033[36m\033[1m{}\033[0m:'.format(utubr))
        for vid in utubrs[utubr]:
            color = '\033[91m'
            if vid[2]:
                color = ''
            if printAll or any(v[0] == vid[0] and v[1] == vid[1] for v in vids):
                print('\t{}\033[1m{}\033[0m) {} - \033[4m{}\033[0m'.format(color, index, vid[0], vid[1]))
            index += 1
        print()

utubrs = readConf()

if args.add:
    utubrs[args.add[0]] = []
    writeConf(utubrs)

if args.update:
    alldaNewStuff = []
    for utubr in utubrs:
        hparse = HTMLParser()
        newstuff = []
        for line in subprocess.run('curl -s https://www.youtube.com/user/{}/videos | grep \'<h3 class="yt-lockup-title ">\''.format(utubr), shell=True, stdout=subprocess.PIPE).stdout.decode('utf-8').split('\n'):
            if line:
                vid = hparse.unescape(re.search('(?<=rel="nofollow">).*(?=</a>)', line).group(0))
                duration = re.search('(?<=Duration: ).*(?=</span>)', line).group(0)
                new = [vid, duration, False]
                if not any(new[0] == vid[0] and new[1] == vid[1] for vid in utubrs[utubr]):
                    newstuff.append(new)
                else:
                    break
        alldaNewStuff += newstuff
        utubrs[utubr] = newstuff + utubrs[utubr]
        # Remember at most 10 items per utubr
        if len(utubrs[utubr]) > 10:
            utubrs[utubr] = utubrs[utubr][0:10]
    if alldaNewStuff:
        print('New stuff emerges!')
        printFancy(alldaNewStuff, False)
    else:
        print('No new vids.')
    writeConf(utubrs)

if args.list:
    printFancy()

if args.watch:
    vid = getVid(args.watch[0])
    utubr = next(utubr for utubr in utubrs if vid in utubrs[utubr])
    vid[2] = True
    writeConf(utubrs)
    subprocess.run('youtube-dl --match-title "{}" -o - https://www.youtube.com/user/{}/videos | mplayer -'.format(vid[0].replace('"', '\\"'), utubr), shell=True)

if args.markAllWatched:
    for utubr in utubrs:
        for vid in utubrs[utubr]:
            vid[2] = True
    writeConf(utubrs)

if args.unwatch:
    vid = getVid(args.unwatch[0])
    vid[2] = False
    writeConf(utubrs)
```