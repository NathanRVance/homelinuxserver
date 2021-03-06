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

parser = argparse.ArgumentParser(description='Play vids made by your favourite creators!')
add = parser.add_argument_group('add', 'Add a creator to be tracked by utube')
add.add_argument('-a', '--add', metavar='NAME', help='Name of the content creator or channel as it appears in their url')
add.add_argument('-s', '--system', choices=['youtube', 'bitchute'], help='Unterlying system to access (default = youtube)', default='youtube')
parser.add_argument('-u', '--update', action='store_true', help='Update listing of videos')
parser.add_argument('-l', '--list', action='store_true', help='List videos available')
parser.add_argument('-w', '--watch', nargs=1, type=int, help='Watch a specific video')
parser.add_argument('-L', '--listen', nargs=1, type=int, help='Listen to a specific video')
parser.add_argument('-W', metavar='URL', help='Watch a specific url')
parser.add_argument('--markAllWatched', action='store_true', help='Mark all videos as watched')
parser.add_argument('--unwatch', nargs=1, type=int, help='Mark a specific video as unwatched')
args = parser.parse_args()

def readConf():
    utubrs = {} # { (utubr, system) : [ [name, duration, hasWatched, url], ...] }
    # Ensure it exists!
    Path(CONF).touch()
    with open(CONF) as f:
        utubr = None
        for line in f.read().split('\n'):
            if not utubr and line:
                utubr = tuple(line.rsplit('&', 1))
                utubrs[utubr] = []
            elif not line:
                utubr = None
            else:
                (vid, line, watched, url) = line.rsplit('&', 3)
                if watched == 'True':
                    watched = True
                else:
                    watched = False
                utubrs[utubr].append([vid, line, watched, url])
    return utubrs

def writeConf(utubrs):
    with open(CONF, 'w') as f:
        for utubr in sorted(utubrs):
            f.write('{}&{}\n'.format(utubr[0], utubr[1]))
            for vid in utubrs[utubr]:
                f.write('{}&{}&{}&{}\n'.format(vid[0], vid[1], vid[2], vid[3]))
            f.write('\n') # blank line

def getVid(index):
    i = 0
    for utubr in sorted(utubrs):
        for vid in utubrs[utubr]:
            if i == index:
                return vid
            i += 1

def getIndex(vid):
    i = 0
    for utubr in sorted(utubrs):
        for v in utubrs[utubr]:
            if v[0] == vid[0] and v[1] == vid[1]:
                return i
            i += 1

def printFancy(vids = [], printAll = True):
    index = 0
    for utubr in sorted(utubrs):
        print('\033[36m\033[1m{}\033[0m:'.format(utubr[0]))
        for vid in utubrs[utubr]:
            color = '\033[91m'
            if vid[2]:
                color = ''
            if printAll or any(v[0] == vid[0] and v[1] == vid[1] for v in vids):
                print('\t{}\033[1m{}\033[0m) {} - \033[4m{}\033[0m'.format(color, index, vid[0], vid[1]))
            index += 1
        print()

def watchUrl(url):
    subprocess.run('youtube-dl -o - {} | mplayer -'.format(url), shell=True)

def listenUrl(url):
    subprocess.run('youtube-dl -o - {} | mplayer -vo null -'.format(url), shell=True)

utubrs = readConf()

if args.add:
    utubrs[(args.add, args.system)] = []
    writeConf(utubrs)

def group(lst, n):
    for i in range(0, len(lst), n):
        val = lst[i:i+n]
        if len(val) == n:
            yield tuple(val)

def downloadBitchute(utubr, count, offset):
    index = []
    for pair in group(subprocess.run('youtube-dl https://www.bitchute.com/channel/{}/ -g -e --max-downloads {} --playlist-start {}'.format(utubr[0], count, offset+1), shell=True, stdout=subprocess.PIPE).stdout.decode('utf-8').split('\n'), 2):
        index.append([pair[0], '0', False, pair[1]])
    return index

def downloadYoutube(utubr):
    index = []
    for userchannel in ['user', 'channel']:
        for line in subprocess.run('curl -s https://www.youtube.com/{}/{}/videos | grep \'<h3 class="yt-lockup-title ">\''.format(userchannel, utubr[0]), shell=True, stdout=subprocess.PIPE).stdout.decode('utf-8').split('\n'):
            if line:
                vid = hparse.unescape(re.search('(?<=rel="nofollow">).*(?=</a>)', line).group(0))
                search = re.search('(?<=Duration: ).*(?=</span>)', line)
                if not search:
                    continue # It's a live stream
                duration = search.group(0)
                url = "https://www.youtube.com" + re.search('(?<=href=").*(?=" rel=)', line).group(0)
                index.append([vid, duration, False, url])
    if len(index) > 10:
        index = index[0:10]
    return index

def isNew(new, utubr):
    return not any(new[0] == vid[0] and new[1] == vid[1] for vid in utubrs[utubr])

def mergeVids(index, utubr):
    merged = []
    for new in index:
        if isNew(new, utubr):
            merged.append(new)
    utubrs[utubr] = merged + utubrs[utubr]
    # Remember at most 10 items per utubr
    if len(utubrs[utubr]) > 10:
        utubrs[utubr] = utubrs[utubr][0:10]
    return merged


if args.update:
    alldaNewStuff = []
    for utubr in utubrs:
        hparse = HTMLParser()
        newstuff = []
        if utubr[1] == 'youtube':
            newstuff = downloadYoutube(utubr)
        elif utubr[1] == 'bitchute':
            # Expensive, so test the waters first
            newstuff = downloadBitchute(utubr, 1, 0)
            if isNew(newstuff[0], utubr):
                newstuff += downloadBitchute(utubr, 9, 1)
        alldaNewStuff += mergeVids(newstuff, utubr)
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
    watchUrl(vid[3])

if args.listen:
    vid = getVid(args.listen[0])
    utubr = next(utubr for utubr in utubrs if vid in utubrs[utubr])
    vid[2] = True
    writeConf(utubrs)
    listenUrl(vid[3])

if args.W:
    watchUrl(args.W)

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
