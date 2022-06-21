---

layout : post
title : HEVC
tags : Video Compression

---

## How des HEVC work?

HEVC is based on the same general structure as previous standards. Source video, consisting of a sequence of video frames, is encoded or compressed by a video encoder to create a compressed video bitstream. The compressed bitstream is stored or transmitted. A video decoder decompresses the bitstream to create a sequence of decoded frames.

The steps carried out by a video encoder(Figure 2) include:

* Partitioning each picture into multiple units.
* Predicting each unit using inter or intra prediction, and substracting the prediction from the unit
* Transforming and quantizing the residual(the difference between the original picture unit and  the prediction)
* Entropy encoding the transform output, prediction information, mode information and headers.

 

A video decoder reverses the steps:

- Entropy decoding and extracting the elements of the coded sequence
- Rescaling and inverting the transform stage
- Predicting each unit and adding the prediction to the output of the inverse transform
- Reconstructing a decoded video image.

![img](https://media-vcodex-com.s3-eu-west-1.amazonaws.com/media/uploads/HEVC%20intro%20/hevc_full_intro_2.png)

### Partitioning 

HEVC supports highly flexible partitioning of a video sequence. Each frame of the sequence is split up into rectangular or square regions (Units or Blocks), each of which is predicted from previously coded data. After prediction, any residual information is transformed and entropy encoded.

Each coded video frame, or picture, is partitioned into Tiles and/or Slices, which are further partitioned into Coding Tree Units (CTUs). The CTU is the basic unit of coding, analogous to the Macroblock in earlier standards, and can be up to 64x64 pixels in siz 

A Coding Tree Unit can be subdivided into square regions known s Coding Units (CUs) using a quadtree structure (Figure 3). Each CU is predicted using Inter or Intra prediction and transformed using one or more Transform Units (see below).

![img](https://media-vcodex-com.s3-eu-west-1.amazonaws.com/media/uploads/HEVC%20intro%20/hevc_full_intro_3.png)

Figure 4 shows a video frame partitioned into slices, with one slice highlighted in blue. The highlighted slice contains six 64x64 CTUs.

![img](https://media-vcodex-com.s3-eu-west-1.amazonaws.com/media/uploads/HEVC%20intro%20/hevc_full_intro_4.png)

Figure 5 shows a close-up of the CTU highlighted in Figure 4. The 64x64 CTU is split into four 32x32 regions, with the top-left 32x32 CU highlighted. In the other four quarters, the 32x32 region is split further, to 16x16 or 8x8 CUs.

![img](https://media-vcodex-com.s3-eu-west-1.amazonaws.com/media/uploads/HEVC%20intro%20/hevc_full_intro_5.png)

###  Prediction

Frames of video are coded using Intra or Inter prediction. Figure 6 shows a sequence of coded video frames or coded pictures. The first picture (0) is coded using Intra prediction only, using spatial prediction from other regions of the same picture. Subsequent pictures are predicted from one, two or more reference pictures, using Inter and/or Intra prediction for each Prediction Unit (PU). The prediction sources for each picture are indicated by arrows.

 ![img](https://media-vcodex-com.s3-eu-west-1.amazonaws.com/media/uploads/HEVC%20intro%20/hevc_full_intro_6.png)