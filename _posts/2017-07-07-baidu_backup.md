---
layout: post
title: linux shell 下用 python 連百度云盤當備份
date: 2017-07-07
tags: 百度云盤 backup
---
百度雲盤的空間極大 ... 2T

用來當備份最好了 ....

保存重要資料請用 DES3 加密 .....
不可能猜的到加密的密碼啦 ......除非太簡單 
請參考 

https://www.echochio.nctu.me/2017/05/Mega_backup/

百度雲盤有些限制 ...單一檔案不能超過 2G

...超過就切割備份 ....(說 百度雲PCS 會自動切沒試過 )

用 pip install bypy 安裝過程如下 : 
```
# pip install requests
Requirement already satisfied: requests in /usr/lib/python2.7/site-packages
# pip install bypy
Collecting bypy
  Downloading bypy-1.5.7-py2.py3-none-any.whl (236kB)
    100% |████████████████████████████████| 245kB 676kB/s
Collecting multiprocess>=0.70.0 (from bypy)
  Downloading multiprocess-0.70.5.zip (1.5MB)
    100% |████████████████████████████████| 1.5MB 795kB/s
Collecting requests-toolbelt>=0.8.0 (from bypy)
  Downloading requests_toolbelt-0.8.0-py2.py3-none-any.whl (54kB)
    100% |████████████████████████████████| 61kB 3.1MB/s
Requirement already satisfied: requests>=2.10.0 in /usr/lib/python2.7/site-packages (from bypy)
Collecting dill>=0.2.6 (from multiprocess>=0.70.0->bypy)
  Downloading dill-0.2.7.tar.gz (64kB)
    100% |████████████████████████████████| 71kB 4.3MB/s
Installing collected packages: dill, multiprocess, requests-toolbelt, bypy
  Running setup.py install for dill ... done
  Running setup.py install for multiprocess ... done
Successfully installed bypy-1.5.7 dill-0.2.7 multiprocess-0.70.5 requests-toolbelt-0.8.0

```

<img src="/images/posts/baidu_backup/p1.jpg">

安裝好了就 bypy list 會連 百度叫你用網頁認證取 token

```
# bypy list
Please visit:
https://openapi.baidu.com/oauth/2.0/authorize?scope=basic+netdisk&redirect_uri=oob&response_type=code&client_id=q8WE4EpC11111gMKNBn
And authorize this app
Paste the Authorization Code here within 10 minutes.
Press [Enter] when you are done
e3f641d24c29111111eefbdb6
Authorizing, please be patient, it may take upto None seconds...
Authorizing/refreshing with the OpenShift server ...
Successfully authorized
/apps/bypy ($t $f $s $m $d):
```
<img src="/images/posts/baidu_backup/p2.jpg">

打入 bypy syncup 上傳 ....去網頁就看到上傳的檔案了

```
# bypy syncup
# ls
client_secret.json  GoUpload.json  main.go  PythonUpload.json  sql.go  up.py
```
<img src="/images/posts/baidu_backup/p3.jpg">

可查看比較
```
# rm -rf *
# bypy -v compare
Loading Hash Cache File '/root/.bypy/bypy.hashcache.json'...
Hash Cache File '/root/.bypy/bypy.hashcache.json' not found, no caching
Gathering local directory ...
Done
Gathering remote directory ...
Done
Comparing ...
Done
==== Same files ===
==== Different files ===
==== Local only ====
==== Remote only ====
F - GoUpload.json
F - main.go
F - client_secret.json
F - up.py
F - PythonUpload.json
F - sql.go

Statistics:
--------------------------------
Same: 0
Different: 0
Local only: 0
Remote only: 6
```

修改單一檔案後再上傳一次 , 或 刪除 都 OK
不知為啥檔案無法下載syncdown ,,,,, 但用網頁下載 OK .....
沒差 ,,,,可刪除或覆蓋就 OK 只是備份資料 ...
如真要那資料就去網頁, 或百度提供的程式 拉下來 ...
(用 -v 可看到比較詳細資訊)

```
# echo test > up.py
# bypy -v syncup
Loading Hash Cache File '/root/.bypy/bypy.hashcache.json'...
Hash Cache File '/root/.bypy/bypy.hashcache.json' not found, no caching
Gathering local directory ...
Done
Gathering remote directory ...
Done
Comparing ...
Done
Deletion request '4346517072182273125' OK
Usage 'list' command to confirm
'up.py' ==> '/apps/bypy/up.py' OK.
# bypy -v rm up.py
Loading Hash Cache File '/root/.bypy/bypy.hashcache.json'...
Hash Cache File loaded.
Deletion request '4346663918813325060' OK
Usage 'list' command to confirm
```

這裡是參數的用法

```
 optional arguments:
  -h, --help            show this help message and exit
  --TESTRUN             Perform python doctest [default: False]
  --PROFILE             Profile the code [default: False]
  -V, --version         show program's version number and exit
  -d, --debug           enable debugging & logging [default: 0]
  -v, --verbose         set verbosity level [default: 0]
  -r RETRY, --retry RETRY
                        number of retry attempts on network error [default: 5
                        times]
  -q QUIT, --quit-when-fail QUIT
                        quit when maximum number of retry failed [default:
                        False]
  -t TIMEOUT, --timeout TIMEOUT
                        network timeout in seconds [default: 60]
  -s SLICE, --slice SLICE
                        size of file upload slice (can use '1024', '2k',
                        '3MB', etc) [default: 20 MB]
  --chunk CHUNK         size of file download chunk (can use '1024', '2k',
                        '3MB', etc) [default: 20 MB]
  -e, --verify          Verify upload / download [default : False]
  -f, --force-hash      force file MD5 / CRC32 calculation instead of using
                        cached values [default: False]
  -l LISTFILE, --list-file LISTFILE
                        input list file (used by some of the commands only
                        [default: None]
  --resume-download RESUMEDL
                        resume instead of restarting when downloading if local
                        file already exists [default: True]
  --include-regex INCREGEX
                        regular expression of files to include. if not
                        specified (default), everything is included. for
                        download, the regex applies to the remote files; for
                        upload, the regex applies to the local files. to
                        exclude files, think about your regex, some tips here:
                        https://stackoverflow.com/questions/406230/regular-
                        expression-to-match-string-not-containing-a-word
                        [default: ]
  --on-dup ONDUP        what to do when the same file / folder exists in the
                        destination: 'overwrite', 'skip', 'prompt' [default:
                        overwrite]
  --no-symlink          DON'T follow symbol links when uploading / syncing up
                        [default: True]
  --disable-ssl-check   DON'T verify host SSL cerificate [default: True]
  -c, --clean           1: clean settings (remove the token file) 2: clean
                        settings and hash cache [default: 0]

Commands:
help command - provide some information for the command
cleancache - remove invalid entries from hash cache file
combine <remotefile> [md5s] [localfile] - try to create a file at PCS by combining slices, having MD5s specified
compare [remotedir] [localdir] - compare the remote direcotry with the local directory
copy/cp <from> <to> - copy a file / dir remotely at Baidu Yun
delete/remove/rm <remotepath> - delete a file / dir remotely at Baidu Yun
downdir <remotedir> [localdir] - download a remote directory (recursively)
downfile <remotefile> [localpath] - download a remote file.
dumpcache - display file hash cache
list/ls [remotepath] [format] [sort] [order] - list the 'remotepath' directory at Baidu PCS
listrecycle [start] [limit] - list the recycle contents
meta <remotepath> [format] - get information of the given path (dir / file) at Baidu Yun.
mkdir <remotedir> - create a directory at Baidu Yun
move/mv/rename/ren <from> <to> - move a file / dir remotely at Baidu Yun
quota/info - displays the quota information
refreshtoken - refresh the access token
restore <remotepath> - restore a file from the recycle bin
search <keyword> [remotepath] [recursive] - search for a file using keyword at Baidu Yun
stream <remotefile> <localpipe> [format] [chunk] - stream a video / audio file converted to M3U format at cloud side, to a pipe.
syncdown [remotedir] [localdir] [deletelocal] - sync down from the remote direcotry to the local directory
syncup [localdir] [remotedir] [deleteremote] - sync up from the local direcotry to the remote directory
upload [localpath] [remotepath] [ondup] - upload a file or directory (recursively)
```
