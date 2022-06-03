# Prores-BitDepth

What is the maximum bitdepth a prores file can be. The Apple documentation is vague on the exact details for each codec and refers to "up to 12 bits" with certain Prores types or supports a certain workflow. https://support.apple.com/en-us/HT202410

Background:

16bits can hold 65536 discreet values. 0-65535
12bits can hold 4096 discreet values. 0-4095
10bits can hold 1024 discreet values. 0-1023

ProRes in an intraframed codec meaning compression is not temporal and any comoression artafacts will spatially in a frame. To avoid this causing issues the mode of the frame. More on this under testing procdure.


To test this I worked in a 16bit RGBA64 space. This was chosen as because I wanted to feed the apple prores encoder every luminance value possible. I am not sure how the apple encoder works with resoect to its "native" input pixel format. I used an AVAssetWriter with a AVAssetWriterInputPixelBufferAdaptor and 'b64a' pixel fornat buffer. For simulated 12 bit I shifted 0-4095 4 bits.

Testing procedure:

Platform M1 Apple MacOS 12.2.1 Objective c and C++ Using AVFoundation.

16bit:
A. Generate 65536 RGBA64 in memory with filled color componenet pixel values of 0-65535 and alpha 65535 where supported.
B. Write the resulting frames in memory to Apple ProRes Types and 16bit Tiff files.
1. Read back created ProRes movie and count unique luminace values in decoded frames.
2. Check that all pixel values within a frame are the same. This is to possible eliminate compression variations between pixels. In

12bit:

Same as 16bit but with 4096 frames represented bit shifted 4bits

Results:
12 Bit :
1.

Apple Encoder and Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes444HQ 
--- | --- | --- | --- |--- |--- |---
Discreet Luminance Values | 876 | 2335 | 3503 | 3503 | 3500 | 3500 

2.
All pixel values with each frame were equal.


16bit :
Apple Encoder and Decoder:
ProRes type | ProRes Proxy | Prores LT | ProRes422 | ProRes422HQ | ProRes4444 | ProRes444HQ 
--- | --- | --- | --- |--- |--- |---
Discreet Luminance Values | 876 | 283 | 290 | 286 | 289 | 3500 





