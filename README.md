# ytdlrc

> Downloads videos and metadata with `youtube-dl` and moves each file on
> completion to an `rclone` remote

`ytdlrc` is a simple shell script wrapper for `youtube-dl` and `rclone`. It
downloads videos and metadata via `youtube-dl` and moves each file on
completion to an `rclone` remote, e.g. Amazon Drive.

* Designed to be executed via cron job.

* Ideal for use on VPS's with small disk space as it moves files on
  completion rather than copying them. In testing I downloaded / uploaded
  ~500GB of videos on a VPS with a 15GB SSD.
  * This can easily be changed to keep the files if you have the disk space;
    see the `rclone_command` variable to do so.

* Includes metadata (xattrs, \*.info.json, \*.description, \*.jpg).

* Reads a file named `snatch.list` to know which URL's / channels /
  playlists you want to download and monitor for new videos.
  * Lines beginning with `#` are ignored.

* Completed video ID's are saved in an file named `archive.list`; this
  prevents `youtube-dl` from re-downloading videos that have already been
  processed and moved to the rclone remote.

* By default, it creates / uses the following structure:

    - `~/ytdlrc/stage`           - download directory
    - `~/ytdlrc/snatch.list`     - list of usernames / URL's to monitor / download
    - `~/ytdlrc/archive.list`    - list of completed video ID's

  These paths can be changed by modifying the `ytdl_*` variables.

* In the download directory, subfolders are created for each line in the
  `snatch.list`. The name of the subfolders depends on whether the processed
  line is a playlist or a channel. This results in the following structure:

    - `~/ytdlrc/stage/Some_Channel_Name/{downloaded files}`
    - `~/ytdlrc/stage/Another_Channel_Name/{downloaded files}`
    - `~/ytdlrc/stage/Some_Playlist_Name/{downloaded files}`
    - `~/ytdlrc/stage/Another_Playlist_Name/{downloaded files}`

  Upon download completion a file is moved to the rclone remote, creating a
  structure such as:

    - `remote:archive/youtube/Some_Channel_Name/{downloaded files}`
    - `remote:archive/youtube/Another_Channel_Name/{downloaded files}`
    - `remote:archive/youtube/Some_Playlist_Name/{downloaded files}`
    - `remote:archive/youtube/Another_Playlist_Name/{downloaded files}`

  The path before the channel / playlist names (`remote:archive/youtube`) is
  set via the `rclone_destination` variable.

* Downloaded files are saved with the following output template:

  `"%(uploader)s.%(upload_date)s.%(title)s.%(resolution)s.%(id)s.%(ext)s"`

  This results in filenames such as:

    - `Channel_Name.20170307.Video_Title.1920x1080.J---aiyznGQ.mp4`
    - `Channel_Name.20170307.Video_Title.1920x1080.J---aiyznGQ.info.json`
    - `Channel_Name.20170307.Video_Title.1920x1080.J---aiyznGQ.description`
    - `Channel_Name.20170307.Video_Title.1920x1080.J---aiyznGQ.jpg`

  See the `ytdl_output_template` variable if you wish to use a different
  output template.

## Example Output

These examples are what you see with `debug=true`.

### Example: Initial execution

```text
[YTDLRC] Lock file doesn't exist. Attempting to create /tmp/ytdlrc.lock...
[YTDLRC] Creating '/tmp/ytdlrc.lock' succeeded. Continuing...
[YTDLRC] Creating download directory: /home/brian/ytdlrc/stage
[YTDLRC] Creating snatch list: /home/brian/ytdlrc/snatch.list
[YTDLRC] Creating archive list: /home/brian/ytdlrc/archive.list
[YTDLRC] Process complete. Removing lock file.
```

### Example: Downloading all videos from ytuser:kurzgesagt

`ytuser:kurzgesagt` is the only line in our `snatch.list`.

#### First Run

```
[YTDLRC] Lock file doesn't exist. Attempting to create /tmp/ytdlrc.lock...
[YTDLRC] Creating '/tmp/ytdlrc.lock' succeeded. Continuing...
[YTDLRC] Processing ytuser:kurzgesagt...
[YTDLRC] Grabbing 'playlist_title' from 'ytuser:kurzgesagt'...
[YTDLRC] 'playlist_title' is 'Uploads_from_Kurzgesagt_In_a_Nutshell'
[YTDLRC] Trimming off 'Uploads_from_' from 'Uploads_from_Kurzgesagt_In_a_Nutshell'...
[YTDLRC] New 'playlist_title' is 'Kurzgesagt_In_a_Nutshell'
[debug] System config: []
[debug] User config: []
[debug] Custom config: []
[debug] Command-line args: ['-4', '--continue', '--download-archive', '/home/brian/ytdlrc/archive.list', '--exec', "/usr/bin/rclone move '{}' 'acd:testing/Kurzgesagt_In_a_Nutshell' --config '/home/brian/.rclone.conf' -v --stats 1s", '--ignore-errors', '--no-overwrites', '--restrict-filenames', '--write-description', '--write-info-json', '--write-thumbnail', '--xattrs', '-f', 'bestvideo+bestaudio', '-o', '/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/%(uploader)s.%(upload_date)s.%(title)s.%(resolution)s.%(id)s.%(ext)s', '--verbose', 'ytuser:kurzgesagt']
[debug] Encodings: locale UTF-8, fs utf-8, out UTF-8, pref UTF-8
[debug] youtube-dl version 2017.03.07
[debug] Python version 3.6.0 - Linux-4.9.11-1-ARCH-x86_64-with-arch
[debug] exe versions: ffmpeg 3.2.4, ffprobe 3.2.4, rtmpdump 2.4
[debug] Proxy map: {}
[youtube:user] kurzgesagt: Downloading channel page
[youtube:playlist] UUsXVk37bltHxD1rDPwtNM8Q: Downloading webpage
[download] Downloading playlist: Uploads from Kurzgesagt – In a Nutshell
[youtube:playlist] playlist Uploads from Kurzgesagt – In a Nutshell: Downloading 58 videos
[download] Downloading video 1 of 58
[youtube] DHyUYg8X31c: Downloading webpage
[youtube] DHyUYg8X31c: Downloading video info webpage
[youtube] DHyUYg8X31c: Extracting video information
[youtube] DHyUYg8X31c: Downloading MPD manifest
[info] Writing video description to: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.description
[info] Writing video description metadata as JSON to: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.info.json
[youtube] DHyUYg8X31c: Downloading thumbnail ...
[youtube] DHyUYg8X31c: Writing thumbnail to: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.jpg
WARNING: Requested formats are incompatible for merge and will be merged into mkv.
[debug] Invoking downloader on 'https://r8---sn-n4v7kne7.googlevideo.com/videoplayback?id=0c7c94620f17df57&itag=299&source=youtube&requiressl=yes&mn=sn-n4v7kne7&mm=31&pl=20&initcwndbps=8067500&mv=m&ms=au&ratebypass=yes&mime=video/mp4&gir=yes&clen=97795996&lmt=1487912067914680&dur=394.266&signature=D4B482B33A060CAD522317EBCF5BA3288C4C95.276EACC1ABBDFFAE143AB016DB9079A47D8329EC&upn=Wj36IpbSSvI&mt=1489800442&key=dg_yt0&ip=104.131.132.15&ipbits=0&expire=1489822135&sparams=ip,ipbits,expire,id,itag,source,requiressl,mn,mm,pl,initcwndbps,mv,ms,ratebypass,mime,gir,clen,lmt,dur'
[download] Destination: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.f299.mp4
[download] 100% of 93.27MiB in 00:01
[debug] Invoking downloader on 'https://r8---sn-n4v7kne7.googlevideo.com/videoplayback?pl=20&ei=Vo3MWIusHurD-APq5JnoDg&clen=6926640&itag=251&gir=yes&upn=TwHyVFumtCo&signature=4ADE59C2CE26DABFB4D6418F61D65F10DF13E25B.1B06F7DE826DD862F39066A6D16B90B12D04EE5D&mime=audio%2Fwebm&initcwndbps=8067500&sparams=clen%2Cdur%2Cei%2Cgir%2Cid%2Cinitcwndbps%2Cip%2Cipbits%2Citag%2Ckeepalive%2Clmt%2Cmime%2Cmm%2Cmn%2Cms%2Cmv%2Cpl%2Crequiressl%2Csource%2Cupn%2Cexpire&ipbits=0&requiressl=yes&keepalive=yes&mn=sn-n4v7kne7&mm=31&mt=1489800442&dur=394.281&id=o-AM978v3HI34Q_CjNzW-0QaneJHqs_vHmTJJzH32vY0-f&lmt=1487796412297530&key=yt6&ip=104.131.132.15&mv=m&source=youtube&ms=au&expire=1489822134&ratebypass=yes'
[download] Destination: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.f251.webm
[download] 100% of 6.61MiB in 00:00
[ffmpeg] Merging formats into "/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.mkv"
[debug] ffmpeg command line: ffmpeg -y -i file:/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.f299.mp4 -i file:/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.f251.webm -c copy -map 0:v:0 -map 1:a:0 file:/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.temp.mkv
Deleting original file /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.f299.mp4 (pass -k to keep)
Deleting original file /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.f251.webm (pass -k to keep)
[metadata] Writing metadata to file's xattrs
[exec] Executing command: /usr/bin/rclone move '/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.mkv' 'acd:testing/Kurzgesagt_In_a_Nutshell' --config '/home/brian/.rclone.conf' -v --stats 1s
2017/03/17 18:29:00 Using RCLONE_CONFIG_PASS password.
2017/03/17 18:29:00 rclone: Version "v1.35-54-gff8f11dβ" starting with parameters ["/usr/bin/rclone" "move" "/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.mkv" "acd:testing/Kurzgesagt_In_a_Nutshell" "--config" "/home/brian/.rclone.conf" "-v" "--stats" "1s"]
2017/03/17 18:29:00 acdp: Saved new token in config file
2017/03/17 18:29:03 Encrypted amazon drive root 'crypt/i04dv4pst54cb73aviioosi6z0/itz12de1esv28z1fob78x6p5q3cp4tw1nboxe4g5u8nswp96qsnt': Modify window not supported
2017/03/17 18:29:03 Encrypted amazon drive root 'crypt/i04dv4pst54cb73aviioosi6z0/itz12de1esv28z1fob78x6p5q3cp4tw1nboxe4g5u8nswp96qsnt': Waiting for checks to finish
2017/03/17 18:29:03 Encrypted amazon drive root 'crypt/i04dv4pst54cb73aviioosi6z0/itz12de1esv28z1fob78x6p5q3cp4tw1nboxe4g5u8nswp96qsnt': Waiting for transfers to finish
2017/03/17 18:29:04 
Transferred:      0 Bytes (0 Bytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        4.1s
Transferring:
 * ...Become_Conscious.1920x1080.DHyUYg8X31c.mkv:  0% done, 0 Bytes/s, ETA: -

2017/03/17 18:29:05 
Transferred:   13.312 MBytes (2.590 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        5.1s
Transferring:
 * ...Become_Conscious.1920x1080.DHyUYg8X31c.mkv: 13% done, 5.278 MBytes/s, ETA: 16s

2017/03/17 18:29:06 
Transferred:   41.312 MBytes (6.707 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        6.1s
Transferring:
 * ...Become_Conscious.1920x1080.DHyUYg8X31c.mkv: 41% done, 6.744 MBytes/s, ETA: 8s

2017/03/17 18:29:07 
Transferred:   68.062 MBytes (9.532 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        7.1s
Transferring:
 * ...Become_Conscious.1920x1080.DHyUYg8X31c.mkv: 68% done, 8.003 MBytes/s, ETA: 3s

2017/03/17 18:29:08 
Transferred:   95.500 MBytes (11.727 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        8.1s
Transferring:
 * ...Become_Conscious.1920x1080.DHyUYg8X31c.mkv: 95% done, 9.293 MBytes/s, ETA: 0s

2017/03/17 18:29:09 
Transferred:   99.754 MBytes (10.913 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        9.1s
Transferring:
 * ...Become_Conscious.1920x1080.DHyUYg8X31c.mkv: 100% done, 9.473 MBytes/s, ETA: 0s

2017/03/17 18:29:21 Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.mkv: Copied (new)
2017/03/17 18:29:21 Kurzgesagt_In_a_Nutshell.20170223.Do_Robots_Deserve_Rights_What_if_Machines_Become_Conscious.1920x1080.DHyUYg8X31c.mkv: Deleted
2017/03/17 18:29:21 
Transferred:   99.754 MBytes (4.598 MBytes/s)
Errors:                 0
Checks:                 1
Transferred:            1
Elapsed time:       21.6s
2017/03/17 18:29:21 Go routines at exit 13
[download] Downloading video 2 of 58
[youtube] RVMZxH1TIIQ: Downloading webpage
[youtube] RVMZxH1TIIQ: Downloading video info webpage
[youtube] RVMZxH1TIIQ: Extracting video information
[youtube] RVMZxH1TIIQ: Downloading MPD manifest
[info] Writing video description to: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.description
[info] Writing video description metadata as JSON to: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.info.json
[youtube] RVMZxH1TIIQ: Downloading thumbnail ...
[youtube] RVMZxH1TIIQ: Writing thumbnail to: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.jpg
WARNING: Requested formats are incompatible for merge and will be merged into mkv.
[debug] Invoking downloader on 'https://r12---sn-n4v7kn76.googlevideo.com/videoplayback?id=455319c47d532084&itag=299&source=youtube&requiressl=yes&mn=sn-n4v7kn76&mm=31&mv=m&initcwndbps=8067500&pl=20&ms=au&ratebypass=yes&mime=video/mp4&gir=yes&clen=96704077&lmt=1486111643921647&dur=416.166&key=dg_yt0&upn=05NaEV1LBgE&signature=35638BDF9D1E278A874D767C7215EF5A9EE8BA23.2CB508CE60975702F17ABD906AC0C079966AD66A&mt=1489800442&ip=104.131.132.15&ipbits=0&expire=1489822162&sparams=ip,ipbits,expire,id,itag,source,requiressl,mn,mm,mv,initcwndbps,pl,ms,ratebypass,mime,gir,clen,lmt,dur'
[download] Destination: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.f299.mp4
[download] 100% of 92.22MiB in 00:02
[debug] Invoking downloader on 'https://r12---sn-n4v7kn76.googlevideo.com/videoplayback?mv=m&source=youtube&ms=au&lmt=1485958101260093&ip=104.131.132.15&key=yt6&mt=1489800442&mn=sn-n4v7kn76&mm=31&id=o-APMAlzJZmlhyM8gJr2Qpia97TJJC-xyNgHdb-x3ftfMO&dur=416.161&gir=yes&clen=7619194&itag=251&ei=cY3MWKenLszL-gP_u4O4Bw&pl=20&initcwndbps=8067500&upn=NJAuw32Dclw&signature=B4CF4BDE47558BAD446C092DD53E8C9E62324A8B.73D00E820A74DD2122C00B502364FCECF99969B4&mime=audio%2Fwebm&requiressl=yes&keepalive=yes&expire=1489822161&sparams=clen%2Cdur%2Cei%2Cgir%2Cid%2Cinitcwndbps%2Cip%2Cipbits%2Citag%2Ckeepalive%2Clmt%2Cmime%2Cmm%2Cmn%2Cms%2Cmv%2Cpl%2Crequiressl%2Csource%2Cupn%2Cexpire&ipbits=0&ratebypass=yes'
[download] Destination: /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.f251.webm
[download] 100% of 7.27MiB in 00:00
[ffmpeg] Merging formats into "/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv"
[debug] ffmpeg command line: ffmpeg -y -i file:/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.f299.mp4 -i file:/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.f251.webm -c copy -map 0:v:0 -map 1:a:0 file:/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.temp.mkv
Deleting original file /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.f299.mp4 (pass -k to keep)
Deleting original file /home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.f251.webm (pass -k to keep)
[metadata] Writing metadata to file's xattrs
[exec] Executing command: /usr/bin/rclone move '/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv' 'acd:testing/Kurzgesagt_In_a_Nutshell' --config '/home/brian/.rclone.conf' -v --stats 1s
2017/03/17 18:29:27 Using RCLONE_CONFIG_PASS password.
2017/03/17 18:29:27 rclone: Version "v1.35-54-gff8f11dβ" starting with parameters ["/usr/bin/rclone" "move" "/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/Kurzgesagt_In_a_Nutshell.20170201.Why_Earth_Is_A_Prison_and_How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv" "acd:testing/Kurzgesagt_In_a_Nutshell" "--config" "/home/brian/.rclone.conf" "-v" "--stats" "1s"]
2017/03/17 18:29:29 Encrypted amazon drive root 'crypt/i04dv4pst54cb73aviioosi6z0/itz12de1esv28z1fob78x6p5q3cp4tw1nboxe4g5u8nswp96qsnt': Modify window not supported
2017/03/17 18:29:29 Encrypted amazon drive root 'crypt/i04dv4pst54cb73aviioosi6z0/itz12de1esv28z1fob78x6p5q3cp4tw1nboxe4g5u8nswp96qsnt': Waiting for checks to finish
2017/03/17 18:29:29 Encrypted amazon drive root 'crypt/i04dv4pst54cb73aviioosi6z0/itz12de1esv28z1fob78x6p5q3cp4tw1nboxe4g5u8nswp96qsnt': Waiting for transfers to finish
2017/03/17 18:29:30 
Transferred:   1.688 MBytes (617.097 kBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        2.8s
Transferring:
 * ...How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv:  1% done, 0 Bytes/s, ETA: -

2017/03/17 18:29:31 
Transferred:   21.062 MBytes (5.513 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        3.8s
Transferring:
 * ...How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv: 21% done, 3.614 MBytes/s, ETA: 21s

2017/03/17 18:29:32 
Transferred:   47.438 MBytes (9.882 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        4.8s
Transferring:
 * ...How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv: 47% done, 4.707 MBytes/s, ETA: 11s

2017/03/17 18:29:33 
Transferred:    79 MBytes (13.613 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        5.8s
Transferring:
 * ...How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv: 79% done, 6.243 MBytes/s, ETA: 3s

2017/03/17 18:29:34 
Transferred:   99.365 MBytes (14.612 MBytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:        6.8s
Transferring:
 * ...How_To_Escape_It.1920x1080.RVMZxH1TIIQ.mkv: 100% done, 7.948 MBytes/s, ETA: 0s
```

This will continue until all videos / metadata are downloaded and moved to
the rclone remote.

#### Second Run

```text
[YTDLRC] Lock file doesn't exist. Creating lock file and continuing...
[YTDLRC] Processing ytuser:kurzgesagt...
[YTDLRC] Grabbing 'playlist_title' from 'ytuser:kurzgesagt'...
[YTDLRC] 'playlist_title' is 'Uploads_from_Kurzgesagt_In_a_Nutshell'
[YTDLRC] Trimming off 'Uploads_from_' from 'Uploads_from_Kurzgesagt_In_a_Nutshell'...
[YTDLRC] New 'playlist_title' is 'Kurzgesagt_In_a_Nutshell'
[debug] System config: []
[debug] User config: []
[debug] Custom config: []
[debug] Command-line args: ['-4', '--continue', '--download-archive', '/home/brian/ytdlrc/archive.list', '--exec', "/usr/bin/rclone move '{}' 'acd:archive/youtube/Kurzgesagt_In_a_Nutshell' --config '/home/brian/.rclone.conf' -v --stats 1s", '--ignore-errors', '--no-overwrites', '--restrict-filenames', '--write-description', '--write-info-json', '--write-thumbnail', '--xattrs', '-f', 'bestvideo+bestaudio', '-o', '/home/brian/ytdlrc/stage/Kurzgesagt_In_a_Nutshell/%(uploader)s.%(upload_date)s.%(title)s.%(resolution)s.%(id)s.%(ext)s', '--verbose', 'ytuser:kurzgesagt']
[debug] Encodings: locale UTF-8, fs utf-8, out UTF-8, pref UTF-8
[debug] youtube-dl version 2017.03.07
[debug] Python version 3.6.0 - Linux-4.9.11-1-ARCH-x86_64-with-arch
[debug] exe versions: ffmpeg 3.2.4, ffprobe 3.2.4, rtmpdump 2.4
[debug] Proxy map: {}
[youtube:user] kurzgesagt: Downloading channel page
[youtube:playlist] UUsXVk37bltHxD1rDPwtNM8Q: Downloading webpage
[download] Downloading playlist: Uploads from Kurzgesagt – In a Nutshell
[youtube:playlist] playlist Uploads from Kurzgesagt – In a Nutshell: Downloading 58 videos
[download] Downloading video 1 of 58
[download] Do Robots Deserve Rights? What if Machines Become Conscious? has already been recorded in archive
[download] Downloading video 2 of 58
[download] Why Earth Is A Prison and How To Escape It has already been recorded in archive
[download] Downloading video 3 of 58
[download] Overpopulation – The Human Explosion Explained has already been recorded in archive
[download] Downloading video 4 of 58
[download] A New History for Humanity – The Human Era has already been recorded in archive
[download] Downloading video 5 of 58
[download] The Most Gruesome Parasites – Neglected Tropical Diseases – NTDs has already been recorded in archive
[download] Downloading video 6 of 58
[download] Fusion Power Explained – Future or Failure has already been recorded in archive
[download] Downloading video 7 of 58
[download] The Most Efficient Way to Destroy the Universe – False Vacuum has already been recorded in archive
[download] Downloading video 8 of 58
[download] How To Eradicate One Of Our Deadliest Enemies – Gene Drive & Malaria has already been recorded in archive
[download] Downloading video 9 of 58
[download] Genetic Engineering Will Change Everything Forever – CRISPR has already been recorded in archive
[download] Downloading video 10 of 58
[download] Death From Space — Gamma-Ray Bursts Explained has already been recorded in archive
[download] Downloading video 11 of 58
[download] What Happened Before History? Human Origins has already been recorded in archive
[download] Downloading video 12 of 58
[download] What Are You? has already been recorded in archive
[download] Downloading video 13 of 58
[download] How Far Can We Go? Limits of Humanity. has already been recorded in archive
[download] Downloading video 14 of 58
[download] Safe and Sorry – Terrorism & Mass Surveillance has already been recorded in archive
[download] Downloading video 15 of 58
[download] Space Elevator – Science Fiction or the Future of Mankind? has already been recorded in archive
[download] Downloading video 16 of 58
[download] The Antibiotic Apocalypse Explained has already been recorded in archive
[download] Downloading video 17 of 58
[download] Why The War on Drugs Is a Huge Failure has already been recorded in archive
[download] Downloading video 18 of 58
[download] The Last Star in the Universe – Red Dwarfs Explained has already been recorded in archive
[download] Downloading video 19 of 58
[download] What Is Something? has already been recorded in archive
[download] Downloading video 20 of 58
[download] Black Holes Explained – From Birth to Death has already been recorded in archive
[download] Downloading video 21 of 58
[download] Quantum Computers Explained – Limits of Human Technology has already been recorded in archive
[download] Downloading video 22 of 58
[download] How Facebook is Stealing Billions of Views has already been recorded in archive
[download] Downloading video 23 of 58
[download] Addiction has already been recorded in archive
[download] Downloading video 24 of 58
[download] What Is Light? has already been recorded in archive
[download] Downloading video 25 of 58
[download] The European Refugee Crisis and Syria Explained has already been recorded in archive
[download] Downloading video 26 of 58
[download] What is Dark Matter and Dark Energy? has already been recorded in archive
[download] Downloading video 27 of 58
[download] What if there was a black hole in your pocket? has already been recorded in archive
[download] Downloading video 28 of 58
[download] The Death Of Bees Explained – Parasites, Poison and Humans has already been recorded in archive
[download] Downloading video 29 of 58
[download] The Fermi Paradox II — Solutions and Ideas – Where Are All The Aliens? has already been recorded in archive
[download] Downloading video 30 of 58
[download] The Fermi Paradox — Where Are All The Aliens? (1/2) has already been recorded in archive
[download] Downloading video 31 of 58
[download] 3 Reasons Why Nuclear Energy Is Terrible! 2/3 has already been recorded in archive
[download] Downloading video 32 of 58
[download] 3 Reasons Why Nuclear Energy Is Awesome! 3/3 has already been recorded in archive
[download] Downloading video 33 of 58
[download] Nuclear Energy Explained: How does it work? 1/3 has already been recorded in archive
[download] Downloading video 34 of 58
[download] Banking Explained – Money and Credit has already been recorded in archive
[download] Downloading video 35 of 58
[download] Measles Explained — Vaccinate or Not? has already been recorded in archive
[download] Downloading video 36 of 58
[download] How Small Is An Atom? Spoiler: Very Small. has already been recorded in archive
[download] Downloading video 37 of 58
[download] The Ultimate Conspiracy Debunker has already been recorded in archive
[download] Downloading video 38 of 58
[download] What Is Life? Is Death Real? has already been recorded in archive
[download] Downloading video 39 of 58
[download] The Ebola Virus Explained — How Your Body Fights For Survival has already been recorded in archive
[download] Downloading video 40 of 58
[download] Is War Over? — A Paradox Explained has already been recorded in archive
[download] Downloading video 41 of 58
[download] Atoms As Big As Mountains — Neutron Stars Explained has already been recorded in archive
[download] Downloading video 42 of 58
[download] Everything You Need to Know About Planet Earth has already been recorded in archive
[download] Downloading video 43 of 58
[download] The Immune System Explained I – Bacteria Infection has already been recorded in archive
[download] Downloading video 44 of 58
[download] Iraq Explained -- ISIS, Syria and War has already been recorded in archive
[download] Downloading video 45 of 58
[download] Are You Alone? (In The Universe) has already been recorded in archive
[download] Downloading video 46 of 58
[download] How to catch a Dwarf Planet -- Triton MM#3 has already been recorded in archive
[download] Downloading video 47 of 58
[download] The Moons of Mars Explained -- Phobos & Deimos MM#2 has already been recorded in archive
[download] Downloading video 48 of 58
[download] How Big is the Moon? MM#1 has already been recorded in archive
[download] Downloading video 49 of 58
[download] Help us make more Videos for Kurzgesagt has already been recorded in archive
[download] Downloading video 50 of 58
[download] Who Invented the Internet? And Why? has already been recorded in archive
[download] Downloading video 51 of 58
[download] The Beginning of Everything -- The Big Bang has already been recorded in archive
[download] Downloading video 52 of 58
[download] Three Ways to Destroy the Universe has already been recorded in archive
[download] Downloading video 53 of 58
[download] The History and Future of Everything -- Time has already been recorded in archive
[download] Downloading video 54 of 58
[download] How The Stock Exchange Works (For Dummies) has already been recorded in archive
[download] Downloading video 55 of 58
[download] The Gulf Stream Explained has already been recorded in archive
[download] Downloading video 56 of 58
[download] Fracking explained: opportunity or danger has already been recorded in archive
[download] Downloading video 57 of 58
[download] The Solar System -- our home in space has already been recorded in archive
[download] Downloading video 58 of 58
[download] How Evolution works has already been recorded in archive
[download] Finished downloading playlist: Uploads from Kurzgesagt – In a Nutshell
[YTDLRC] Process complete. Removing lock file.
```

All available videos were already downloaded and moved to the rclone remote,
so there was nothing to be done on the second run other than check for new
uploads.

## Installation

1. [Download](https://github.com/bardisty/ytdlrc/archive/master.zip) or clone the repository:

   `git clone https://github.com/bardisty/ytdlrc`

2. `cd` into the directory:

   `cd ytdlrc`

3. Open `ytdlrc` in your text editor and see [Usage](#usage).

## Usage

1. Modify the `rclone_destination` variable with your rclone remote
   destination path (default: `remote:archive/youtube`).

2. Run the script once to generate the working directory, download
   directory, snatch list, and archive list.

3. Put the URL's / channels / playlists you want to download in the
   `snatch.list` file, one per line, e.g.:
      - `ytuser:username`
      - `https://www.youtube.com/user/username`
      - `https://www.youtube.com/playlist?list=PLK9Sc5q_4K6aNajVLKtkaAB1JGmKyccf2`

4. (Optional) Run the script once or twice with debugging enabled to ensure
   everything is okie dokie; disable debugging when done.

5. Set up a cron job to execute the script hourly or however often you want.
      - If you set up the cron job by moving the `ytdlrc` file to one of the
        `/etc/cron.*/` directories, you may want to modify the
        `rclone_config` variable with the path to your rclone config
        (default: `$HOME/.rclone.conf`), otherwise it will look inside
        `/root` for your config file. You may also want to modify the
        `ytdl_root_dir` variable so runtime files aren't created inside
        `/root`.

## Requirements

* [coreutils](https://www.gnu.org/software/coreutils/coreutils.html)
* [rclone](http://rclone.org/)
* [youtube-dl](https://rg3.github.io/youtube-dl/)

## License

[MIT](LICENSE)

