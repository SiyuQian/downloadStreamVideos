# downloadStreamVideos
This is a simple tutorial for teaching you how to download videos from websites for free.


## Background
I have helped my girlfriend to download all the videos for **The Rap of China** from a website. Although the website allows us to watch the video online, there is no option for downloading the video. My girlfriend wanted to store this classic TV show on her laptop's hard drive so it is time for me to show her how programming helps people !!! LOL.

The whole process can be summarized as **three section**:
1. Find the actual video source links
2. Download all .ts files
3. Convert .ts to .mp4 file


## Find the actual video source links

Locate the embed player in the HTML source by using your web browser's developer tool. If you can find some URL is linking to some file called 'xxxx.m3u8'. Great, you probably should carry on to read this article.

If you are lucky, you would be able to find some URL links which direct you to some files with .ts as the file extension. I am not that lucky, in my case, the xxx.m3u8 file contains another xxx.m3u8 link so I have to keep digging the **true** resource links to the video.

Exmaple URL links:
```
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161000.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161001.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161002.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161003.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161004.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161005.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161006.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161007.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161008.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161009.ts
http://example.com/20170720/eYjHWloB/600kb/hls/EF2L1161010.ts
```


## Download all .ts files

There is a lot of ways which could achieve this. As you already have all the links for downloading a bunch of .ts files for merging as the target video. In my case, there are near two thousands of URL links which have to be downloaded. The download software usually is to heavy for this job or have some limitations with the max tasks number.

So I have wrote a **PHP script** to download all the files, here is the code:

``` php
#!/usr/bin/php
<?php
function downloadFile($url, $toDir, $withName) {
	// remove special chars in the URL
	$url = str_replace(PHP_EOL, '', $url);
    // open file in rb mode
    if ($remoteFile = @fopen($url, 'rb')) {
    	if (!$remoteFile) {
		   throw new Exception("Error: " . $http_response_header[0]);
		}
        // local filename
        $localFile = $toDir . "/" . $withName;
         // read buffer, open in wb mode for writing
        if ($localFileHandle = fopen($localFile, 'wb')) {
            // read the file, buffer size 8k
            while ($buffer = fread($remoteFile, 8192)) {
                // write buffer in  local file
                fwrite($localFileHandle, $buffer);
            }
            // close local
            fclose($localFileHandle);
        } else {
            // could not open the local URL
            fclose($remoteFile);
            return false;
        }
        // close remote
        fclose($remoteFile);
        return true;
    } else {
    	var_dump($url);
    	var_dump($http_response_header);
        // could not open the remote URL
        return false;
    }
} // end

$counter = 0;
$handle = fopen("/home/siyu/Downloads/aiqiyi.txt", "r");
if ($handle) {
    while (($line = fgets($handle)) !== false) {
        // process the line read.
        if ($line) {
        	$urlParts = explode('/', $line);
        	$fileName = end($urlParts);
        	// if file already exists, do not download it again
        	if (file_exists('/home/siyu/Downloads/aiqiyi'. DIRECTORY_SEPARATOR . $fileName)) {
        		echo "#$counter: Skipped. Already exists." . PHP_EOL;
        		$counter++;
        		continue;
        	}

        	if (downloadFile($line, '/home/siyu/Downloads/aiqiyi', $fileName)) {
        		echo "#$counter: Downloaded" . PHP_EOL;
        	} else {
        		echo "#$counter: Failed" . PHP_EOL;
        	}
        }
        $counter++;
    }
    fclose($handle);
} else {
    // error opening the file.
    throw new Exception('The file which contains all source links is not available for read.');
}
```
For running the code above, you would have to save all the source links into a .txt file and replace '/home/siyu/Downloads/aiqiyi.txt' with your path and file name for the source links.

Also, replace the target folder as well, '/home/siyu/Downloads/aiqiyi' to your ${TARGET_FOLDER}.

**NOTE: Sometimes a few files might be failed to be downloaded. Re-run the script until all row shows #$counter: Skipped. Already exists.**


## Convert .ts to .mp4 file
Ok. Eventually that we have all the .ts files in our target folder.

Please save the following bash as **export.sh**:
``` bash
#!/bin/bash
for file in *.ts
do
  echo file ${file} >> mylist.txt
done
```

then, you would get a file called mylist.txt which contains all the content of the .ts files.

Install ffmpeg if you haven't. 
``` bash
sudo apt-get update
sudo apt-get install ffmpeg
```
Run the following command to merge all content in the mylist.txt file to all.ts:
``` bash
ffmpeg -f concat -i mylist.txt -c copy all.ts
```

Run the following command to convert .ts file to .mp4 file:
``` bash
ffmpeg -i all.ts -c:v libx264 -c:a copy -bsf:a aac_adtstoasc output.mp4
```

## Ok. All done. Enjoy the video!!! :)
