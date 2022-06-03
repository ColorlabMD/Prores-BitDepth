# Prores-BitDepth

What is the maximum bitdepth a prores file can be. The Apple documentation is vague on the exact details for each codec and refers to "up to 12 bits" with certain Prores types or supports a certain workflow. https://support.apple.com/en-us/HT202410

Background:

16bits can hold 65536 discreet values. 0-65535
12bits can hold 4096 discreet values. 0-4095
10bits can hold 1024 discreet values. 0-1023

ProRes in an intraframed codec meaning compression is not temporal and any comoression artafacts will spatially in a frame. To avoid this causing issues the mode of the frame. More on this under testing procdure.


To test this I worked in a 16bit RGBA64 space. This was chosen as because I wanted to feed the apple prores encoder every luminance value possible. I am not sure how the apple encoder works with resoect to its "native" input pixel format. I used an AVAssetWriter with a AVAssetWriterInputPixelBufferAdaptor and 'b64a' pixel fornat buffer. The following tests were performed in 

Testing procedure:





Results:



