# Prores-BitDepth

What is the maximum bitdepth a prores file can be? The Apple documentation is vague on the exact details for each codec and refers to "up to 12-bit" with certain Prores types or supports a certain workflow. https://support.apple.com/en-us/HT202410 Apple Quicktime Player info screen indicates "up to 12-bit" on all variants. ![alt text](https://github.com/ColorlabMD/Prores-BitDepth/blob/main/qtplayerinfo.png)


## Background:

16bits can hold 65536 discreet values. 0-65535

12bits can hold 4096 discreet values. 0-4095

10bits can hold 1024 discreet values. 0-1023

ProRes is an intraframe codec meaning compression is not temporal and any compression artifacts will spatially in a frame.
ProRes Types:
 ProRes Proxy , Prores LT , ProRes422 , ProRes422HQ , ProRes4444 , ProRes4444XQ 

To test this the graphics used a 16bit per component RGBA space. This was chosen to feed the Apple ProRes encoder every luminance value possible. I am not sure how the apple encoder works with respect to its "native" input pixel format. I used an AVAssetWriter with a AVAssetWriterInputPixelBufferAdaptor and 64bit RGBA pixel format buffer. For 12 and 10 bit I shifted 4 and 6bits respectively.

FFmpeg was also used, however the FFmpeg encoder prores_ks only accpets 10 bit pixel buffer format. The decoder offers 12bit output but only for the 444 variants.


## Files:
All the tested ProRes outputs are provided as are the Tiff files for 12 bit and 10bit. If anyone would like receive the 16bit tiff sequence feel free to ask. 
The mac os program to test for the values created from the encoded sequence is provided as a commandline tool movdepthcheck.

Usage: movdepthcheck input.mov

arguments: -h only checks the top half of the image. (height/2)


## Testing procedure:

Platform: M1 Apple MacOS 12.2.1 Objective C and C++ Using AVFoundation.
Images were created as 16bit per component RGBA at a resolution of 720x480

Apple Encoder
16bit:
A. Generate 65536 RGBA64 in memory with filled color component pixel values of 0-65535 and alpha 65535 where supported.
B. Write the resulting frames in memory to Apple ProRes Types and 16bit Tiff files. (Tiff files are used for FFmpeg encoding)
1. Read back created ProRes movie and count unique luminance values(or red values in red only frames) in decoded frames.
2. Check that all pixel values within a frame are the same. This is used to eliminate unique pixel values caused by compression variations between pixels spatially. (After testing, all files were uniform pixel values at all locations in all files generated.)
12bit:
Same as 16bit but with 4096 frames represented bit shifted 4 bits
10bit:
Same as 16bit but with 1024 frames represented bit shifted 6 bits

Apple Decoder
1. Read frames to 16bit per component buffer via ...
2. Check all values in frame are equal
3. For Luminance frames check R=G=B
4. For R only color frames check G=B=0 (It does not in any of the formats.  However G,B do not change with out R. R is always ascending while G,B fluctuate between ascending and descending at a small amount in 1 or 2 of the LSBs)

FFmpeg Encoder

FFmpeg 5.0.1  libavcodec 59. 18.100 prores_ks
FFmpeg command for the 16bit/component tiff files profiles 0-5:  ffmpeg -f image2 -framerate 24 -i /input_%05d.tiff -c:v prores_ks -profile (0-5) output.mov

FFmpeg Decoder

ffmpeg -i input.mov -pix_fmt rgb48be output_%05d.tiff or for quick check ffmpeg -i input.mov -f framemd5.md5 and check for duplicates.

In case the encoder had a special case for the entire frame being a single value, I also created files with the top half of the screen having the test values and the bottom half having randomly generated noise. This did not have any effect on the results. These files are avaible upon request but can not be zipped to fit under the github max file size limit.

## Results:

Apple Encoder and Apple Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes4444XQ 
--- | --- | --- | --- |--- |--- |---
16 bit Discreet Luminance Values | 877 | 2337 | 3505 | 3505 | 3505 | 3505 
12 bit Discreet Luminance Values | 877 | 2336 | 3504 | 3504 | 3504 | 3504 
10 bit Discreet Luminance Values | 876 | 1024 | 1024 | 1024 | 1024 | 1024 
16 bit Discreet Red Only Values | 708 | 1880 | 2806 | 2806 | 2814 | 2814 
12 bit Discreet Red Only Values | 682 | 1689 | 2381 | 2381 | 2385 | 2385 
10 bit Discreet Red Only Values | 594 | 993 | 1024 | 1024 | 1024 | 1024 

FFmpeg prores_ks Encoder and Apple Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes4444XQ 
--- | --- | --- | --- |--- |--- |---
12 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 877 | 877 
10 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 877 | 877 
12 bit Discreet Red Only Values | 685 | 685 | 685 | 685 | 685 | 685 
10 bit Discreet Red Only Values | 589 | 589 | 589 | 589 | 589 | 589 

Apple Encoder and FFmpeg Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes4444XQ 
--- | --- | --- | --- |--- |--- |---
12 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 3504 | 3504 
10 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 1024 | 1024
12 bit Discreet Red Only Values | 809 | 808 | 808 | 808 | 2635 | 2635 
10 bit Discreet Red Only Values | 656 | 656 | 660 | 660 | 1024 | 1024 

## Conclusions:

1. ProRes is truly "up to 12bit" in all formats except ProRes proxy. "up to 12 bits" meaning more than 11bits(2048) and less than 4096. 
2. ProRes decoding requires 12bits to store all possible output values except in ProRes proxy.
3. All formats, except ProRes proxy, can produce 1024 discreet output values from 1024 input values.(Full 10bit)
4. Color bitdepth is lower than luminance even though the name of the codec ProRes444. "4:4:4" commonly means sample ratios between components but it would seem this is not the case with ProRes.
5. Beware of using the FFmpeg prores_ks encoder and decoder.(except decoding 444 files where Apple and FFmpeg are very comprable) 





