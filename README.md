# Prores-BitDepth

What is the maximum bitdepth a prores file can be. The Apple documentation is vague on the exact details for each codec and refers to "up to 12 bits" with certain Prores types or supports a certain workflow. https://support.apple.com/en-us/HT202410

Background:

16bits can hold 65536 discreet values. 0-65535

12bits can hold 4096 discreet values. 0-4095

10bits can hold 1024 discreet values. 0-1023

ProRes in an intraframed codec meaning compression is not temporal and any compression artafacts will spatially in a frame. To avoid this causing issues the mode of the frame. More on this under testing procdure.
ProRes Types:
 ProRes Proxy , Prores LT , ProRes422 , ProRes422HQ , ProRes4444 , ProRes4444XQ 



To test this I worked in a 16bit RGBA64 space. This was chosen as because I wanted to feed the apple prores encoder every luminance value possible. I am not sure how the apple encoder works with respect to its "native" input pixel format. I used an AVAssetWriter with a AVAssetWriterInputPixelBufferAdaptor and 64bit RGBA pixel format buffer. For simulated 12 and 10 bit I shifted 4 and 6bits respectivly.

FFMPEG was also used, however the ffmpeg encoder prores_ks only accpets 10 bit pixel buffer format.

## Testing procedure:

Platform M1 Apple MacOS 12.2.1 Objective C and C++ Using AVFoundation.
Images were created as 16bit per component RGBA at a resolution of 720x480

Apple Encoder
16bit:
A. Generate 65536 RGBA64 in memory with filled color componenet pixel values of 0-65535 and alpha 65535 where supported.
B. Write the resulting frames in memory to Apple ProRes Types and 16bit Tiff files. (Tiff files are used for FFMPEG encoding)
1. Read back created ProRes movie and count unique luminace values(or red values in red only frames) in decoded frames.
2. Check that all pixel values within a frame are the same. This is to possible eliminate compression variations between pixels.
12bit:
Same as 16bit but with 4096 frames represented bit shifted 4 bits
10bit:
Same as 16bit but with 1024 frames represented bit shifted 6 bits

Apple Decoder
1. Read frames to 16bit per compenent buffer via ...
2. Check all values in frame are equal
3. For Luminance frames check R=G=B
4. For R only color frames check G=B=0 (It does not in any of the formats.  However G,B do not change with out R. R is always ascending while G,B flucuate between ascending and descending at a small amount in 1 or 2 of the LSBs)

FFMPEG Encoder

FFMPEG 5.0.1  libavcodec 59. 18.100 prores_ks
FFMPEG command for the 16bit/componenet tiff files profiles 0-5:  ffmpeg -f image2 -framerate 24 -i /input_%05d.tiff -c:v prores_ks -profile (0-5) output.mov

FFMPEG Decoder

ffmpeg -i input.mov -pix_fmt rgb48be output_%05d.tiff



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

FFMPEG prores_ks Encoder and Apple Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes4444XQ 
--- | --- | --- | --- |--- |--- |---
12 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 877 | 877 
10 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 877 | 877 
12 bit Discreet Red Only Values | 685 | 685 | 685 | 685 | 685 | 685 
10 bit Discreet Red Only Values | 589 | 589 | 589 | 589 | 589 | 589 

Apple Encoder and FFMPEG Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes4444XQ 
--- | --- | --- | --- |--- |--- |---
12 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 3504 | 3504 
10 bit Discreet Luminance Values | 877 | 877 | 877 | 877 | 1024 | 1024
12 bit Discreet Red Only Values | 809 | 808 | 808 | 808 | 2635 | 2635 
10 bit Discreet Red Only Values | 656 | 656 | 660 | 660 | 1024 | 1024 






