# HM-16.14

## Complexity Reduction on Fast Intra Mode Algorithm

![HEVC RDO Block Diagram](http://video.ee.ntu.edu.tw/~tcm/HEVC/RDO.PNG)
Image from [6]

In HEVC, there are total 35 modes in Intra-pitcure prediction, including 33 angular mode, planar mode, and DC mode. 
More intra mode can predict more accurate result so that the distortion between reconstructed and source pixels decrease.
However, if all the intra modes were the candidates for RDO, the complexity of video encoder is impratical for most applications. 
In order to reduce the computational complexity of encoder, the fast intra mode decision algorithm is adopted in HEVC Reference software, HM. 
In HM, the fast intra mode decision algorithm has two parts. 
The first part predicts the pixel in current Coding Unit (CU) from the neighboring reconstructed CU. 
All angular mode was performed by fast intra algorithm to generate predicted pixel and then generating the residual with reconstructed pixels.
The second part is the two dimension 8x8, 4x4, and 2x2 Hadamard transform. 

Despite the fast intra algorithm is adopted in HM, it is not part of HEVC standard. 
Therefore, some fast intra algorithms are proposed to reduce the complexity of encoder.
For example, Shen et al. [1] proposed a fast CU size decision and mode decision algorithm from the previous coding result of neighboring CU.
In [2], the fast CU slitting and pruning decision based on the Bayes decision rule was proposed. 
Some other approaches reduces the number of intra mode candidates by exploring the characteristics of local image. 
In the previous video coding technique, H.264, one approach [3] is based on the directionalities of the source and neighboring CU.
Then in HEVC, same approach is also proposed to reduce the complexity of encoder [4] on 2012.
Since the fast intra algorithm is encoder issue, there are more agressive method to reduce hardware requirement and parallel processing.
For instance, the intra prediction requires the reconstructed frame to predict current CU. 
In order to parallel processing in each level of CU, the fast intra can use the source pixel to predict current CU without encoder-decoder mismatch while it will suffer more video quality drop.

While working in MediaTek, I was one of the video encoder team and in charge of designing the intra prediction engine.
In 2012, our team aimed on a real-time 30fps 4kx2k HEVC encoder even before the HEVC standard was finialized.
In hardware design, one of the most important issue is the trade-off between compuational complexity and video quality.
The reasonable video quality drop was compromised to the reduction of complexity and chip area.
Therefore, the fast intra algorithm is one of the choice to release the burden of computation and cost since it is not part of standard. 
The reduction of computation not only saves area cost but also decreases the total processing cycle per CU to achieve real-time requirement.

First, I implement directional approaches from [3, 4] to reduce the possible candidates of fast intra algorithm in HM 16.14.

The encoding configuration is all intra, qp 32 (as default).

In brief, gradient method can reduce overall intra encoding time 18.34% on average while PSNR is down -0.07dB.
The bit rate increases about 2.3%.

The detail results are following. 

| Sequence         | Image Size | BD rate  (%)| BD PSNR Drop (dB)| Time Reduction %|
| ---------        | -----------|-------------    | -------------    |---------------- |
| Traffic          | 2560x1600  |  +2.146         | -0.0607          | 17.95           |
| PeopleOnStreet   | 2560x1600  |  +3.165         | -0.125           | 19.91           |
| Kimono           | 1920x1080  |  +1.015         | -0.0319          | 17.96           |
| ParkScene        | 1920x1080  |  +0.788         | -0.0667          | 16.23           |
| Cactus           | 1920x1080  |  +1.914         | -0.0539          | 17.56           |
| BasketballDrive  | 1920x1080  |  +3.095         | -0.0327          | 19.47           |
| BQTerrace        | 1920x1080  |  +1.424         | -0.0539          | 18.21           |
| BasketballDrill  | 832x480    |  +2.175         | -0.0693          | 19.21           |
| BQMall           | 832x480    |  +2.746         | -0.0799          | 18.41           | 
| PartyScene       | 832x480    |  +2.235         | -0.1067          | 18.04           | 
| BasketballPass   | 416x240    |  +3.298         | -0.0812          | 17.31           | 
| BQSquare         | 416x240    |  +3.159         | -0.118           | 17.02           | 
| BlowingBubble    | 416x240    |  +2.300         | -0.0993          | 17.40           | 
| Vidyo1           | 1280x720   |  +2.852         | -0.0775          | 19.76           | 
| Vidyo3           | 1280x720   |  +1.906         | -0.0619          | 19.61           | 
| Vidyo4           | 1280x720   |  +2.665         | -0.0584          | 19.35           | 
| Average          |            |  +2.305         | -0.0735          | 18.34           | 

![RD curve](https://github.com/b93901190/HM-16.14/blob/master/result/RD_curve.PNG)

Second, we can also subsample the residual of CU to reduce the number of adder required in Hadamard Transform.
As in [5], the checker board subsample method only requires 50% area cost. 
In my implementation, the number of adder in 8x8 Hadamard Transform is reduced from 448 to 192, around 57% reduction. 
According to synthesis report, 1 bit full adder occupies 9.5 gate-count in average.
If the each residual is 16-bit, the total area saving is around 28k gate-count.
From the chip result [6], the total gate-count of intra-picture module is 1148k. 
Hence, subsampling the residual of CU in fast intra contributes 2.5% area reduction.
But the 2.5% reduction only consider the adders. More reduction can be achieved since the required area of register and SRAM is lower than w/o subsampling of the residual.

The simulation result of gradient fast intra algorithm and subsample of residual is following.

| Sequence         | Image Size | BD rate    (dB)| BD PSNR Drop (dB)| Time Reduction %|
| ---------        | -----------|-------------   | -------------    |---------------- |
| Traffic          | 2560x1600  | +2.128         | -0.0619          | 18.470          |
| PeopleOnStreet   | 2560x1600  | +3.194         | -0.0741          | 19.125          |
| Kimono           | 1920x1080  | +0.997         | -0.0324          | 18.651          |
| ParkScene        | 1920x1080  | +0.765         | -0.0674          | 17.164          |
| Cactus           | 1920x1080  | +1.909         | -0.0553          | 18.230          |
| BasketballDrive  | 1920x1080  | +3.096         | -0.0334          | 20.121          |
| BQTerrace        | 1920x1080  | +1.426         | -0.0546          | 18.891          |
| BasketballDrill  | 832x480    | +2.206         | -0.0717          | 19.195          |
| BQMall           | 832x480    | +2.743         | -0.0822          | 19.050          | 
| PartyScene       | 832x480    | +2.231         | -0.1121          | 17.774          | 
| BasketballPass   | 416x240    | +3.400         | -0.0808          | 18.294          | 
| BQSquare         | 416x240    | +3.199         | -0.1242          | 18.128          | 
| BlowingBubble    | 416x240    | +2.248         | -0.1061          | 18.230          | 
| Vidyo1           | 1280x720   | +2.803         | -0.0799          | 20.238          | 
| Vidyo3           | 1280x720   | +1.926         | -0.0626          | 20.456          | 
| Vidyo4           | 1280x720   | +2.687         | -0.059           | 20.284          | 
| Average          |            | +2.310         | -0.0723          | 18.894          | 

As the result above shown, the subsampling on Hadamard transform in fast intra algorithm does not deteriorate much in terms of BD rate and processing time.
Therefore, it can reduce the area require without sacrificing video quality in this simulation.

If a more agressive way to reduce are cost is needed, the pre and post-filtering in angular prediction can be removed inorder to do subsample on current CU in fast intra. 
But since it is not for software encoder, I put this in future work.

All the HM-16.14 simulation result is in [here](https://github.com/b93901190/fast_intra/tree/master/result)


## HM Simulation

To turn on Gradient fast intra, do not forget to turn the original fast intra off.

In TLibCommon/TypeDef.h, set GRADIENT_FAST_INTRA 1/0 to on/off gradient fast intra

In TLibCommon/TypeDef.h, set HAD_SUBSAMPLE 1/0 to on/off subsample on Hadamard transform

To turn on/off the original fast intra, set HM_FAST_INTRA 1/0

## Reference

[1] Shen, Liquan, Zhaoyang Zhang, and Ping An. "Fast CU size decision and mode decision algorithm for HEVC intra coding." IEEE Transactions on Consumer Electronics 59.1 (2013): 207-213.

[2] Cho, Seunghyun, and Munchurl Kim. "Fast CU splitting and pruning for suboptimal CU partitioning in HEVC intra coding." IEEE Transactions on Circuits and Systems for Video Technology 23.9 (2013): 1555-1564.

[3] Pan, Feng, et al. "Fast mode decision algorithm for intraprediction in H. 264/AVC video coding." IEEE Transactions on Circuits and Systems for Video Technology 15.7 (2005): 813-822.

[4] Jiang, Wei, Hanjie Ma, and Yaowu Chen. "Gradient based fast mode decision algorithm for intra prediction in HEVC." Consumer Electronics, Communications and Networks (CECNet), 2012 2nd International Conference on. IEEE, 2012.

[5] Huang, Yu-Wen, et al. "Analysis, fast algorithm, and VLSI architecture design for H. 264/AVC intra frame coder." IEEE Transactions on Circuits and systems for Video Technology 15.3 (2005): 378-401.

[6] Tsai, Sung-Fang, Cheng-Han Tsai, and Liang-Gee Chen. "Encoder Hardware Architecture for HEVC." High Efficiency Video Coding (HEVC). Springer, Cham, 2014. 343-375.
