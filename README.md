# Gipuma


Source code for the paper:

S. Galliani, K. Lasinger and K. Schindler, [Massively Parallel Multiview Stereopsis by Surface Normal Diffusion](http://www.prs.igp.ethz.ch/content/dam/ethz/special-interest/baug/igp/photogrammetry-remote-sensing-dam/documents/pdf/galliani-lasinger-iccv15.pdf), ICCV 2015

## Authors
- [Silvano Galliani](mailto:silvano.galliani@geod.baug.ethz.ch)
- [Katrin Lasinger](mailto:katrin.lasinger@geod.baug.ethz.ch)


**IMPORTANT**: If you use this software please cite the following in any resulting publication:
```
@InProceedings{Galliani_2015_ICCV,
     author  = {Galliani, Silvano and Lasinger, Katrin and Schindler, Konrad},
     title   = {Massively Parallel Multiview Stereopsis by Surface Normal Diffusion},
     journal = {The IEEE International Conference on Computer Vision (ICCV)},
     month   = {June},
     year    = {2015}
}
```

## Requirements:
 - [Cuda](https://developer.nvidia.com/cuda-downloads) >= 6.0
 - Nvidia video card with compute capability at least 3.0, see https://en.wikipedia.org/wiki/CUDA#Supported_GPUs
 - [Opencv](http://opencv.org) >= 2.4
 - [cmake](http://cmake.org)

## Tested Operating Systems
 - Ubuntu GNU/Linux 14.04 (use their repository for cuda sdk and nvidia drivers)
 - Windows with Visual Studio 2012/2013 (it's working directly with cmake)

## How to compile it
Use cmake for both Windows and Linux.
For linux it gets as easy as:
```bash
cmake .
make
```

## How does it work?
Gipuma itself is only a matcher. It will compute a depthmap with respect to the specified reference camera.

For each camera gipuma computes the _noisy_ depthmap. The final fusion of depthmap is obtained with [fusibile](https://github.com/kysucix/fusibile)

## Examples
 ### Middlebury
 inside gipuma directory first download middlebury data
 ```bash
 ./scripts/download-middlebury.sh
 ./scripts/download-middlebury.sh
 ```
 Then run the model you prefer. For example for the dino sparse ring:
 ```bash
 ./scripts/dinoSparseRing.sh
 ```
 It will fuse the dephmaps without considering point projecting on the image with an intensity lower than 20 (on a scale 0-255). The result should match the  middlebury benchmark submission (excluding [Poisson reconstruction](http://www.cs.jhu.edu/~misha/Code/PoissonRecon/))


### DTU dataset
http://roboimagedata.compute.dtu.dk/?page_id=24

In case you did not download it, you can download the sample dataset (warning: 6GB of data)):
```bash
./scripts/download-dtu.sh
```
Then to reconstruct image 1 using fast settings:
```
./scripts/dtu_fast.sh 1
```
or for the accurate version
```
./scripts/dtu_accurate.sh 1
```
In case you need to reconstruct other images just copy the corresponding folder inside ./data/dtu/SampleSet/MVS\ Data/Rectified/

### Strecha dataset
## Configuration

### Necessary Parameters
The minimum information Gipuma needs is camera information and image list
Gipuma relies on known camera information. You can provide this information in 3 different ways:

| Parameter Name | Syntax | Comment |
| -------------- | --------------------------------- | --------------- |
| pmvs_folder | -pmvs_folder \<folder\> | The easiest way is to point gipuma to the output of VisualSFM for pmvs. Images will be taken from \<pmvs\>/visualize and cameras from \<pmvs\>/txt/ Additionally 3d points in \<pmvs\>/bundle.rd.out |
| krt_file | -krt_file \<file\> | In this way camera information is read from a file as specified by Middlebury benchmark http://vision.middlebury.edu/mview/data/ |
| p_folder | -p_folder \<folder\> | This parameter expects a folder with a list of textfiles containing the P matrix on 3 lines and the same filename as the images but with ".P" appended |

To specify an image list in case a pmvs folder is not specify a list of filename is needed with an image folder.
For example:
```
./gipuma image1.jpg image2.jpg -img_folder images/ -krt_file Temple.txt
```

### Important Parameters
 **gipuma** comes with a good-enough parameter sets, but to obtain best results some setting can be optimized


| Parameter Name | Syntax and default Value | Comment |
| -------------- | ------------------------ | --------------- |
| camera_idx | --camera_idx=00 | This value set the reference camera to be used. The resulting depth map will be computed with respect to this camera |
| blocksize | --blocksize=19 | It's an important value that affects the patch size used for cost computation. Its value is highly dependent on the image resolution and the object size. Suggested value range from 9 for Middlebury size image (640x480) to 25 for DTU dataset (1600x1200) |
| iterations | --iterations=8 | This parameter controls the amount of normal diffusion employed for the reconstruction. A value bigger than 8 rarely improves the reconstruction. Recuding its value trades-off runtime for quality of reconstruction |
| min_angle and max_angle | --min_angle=5 --max_angle=45 | The reference camera will be matched with respect to other cameras that have a intersection angle of the principal ray withing the specified range. For datasets with many images a range 10-30 degree is suggested. For dataset with a sparse set of images (as middlebury SparseRing) a bigger range is needed |
| max_views | --max_views=9 | In case more than max_views cameras survive the angle selection, a random set of max_views cameras is considered. |
| depth_min and depth_max | --depth_min=-1 --depth_max=-1 | This value set the minimum depth for the reconstruction in world coordinate. In case it is not set it is computed as the minimum and maximum range for all the camera from the 3d points inside the specified pmvs directory. In case only a list of P matrices is given, it is computed from the minimum and maximum range of depth obtained when setting the viewing angle to the minimum and maximum specified |

### Optional Parameters
 The following parameters _can_ be tweaked at will but in our experience they do not affect the overall reconstruction
 TODO

## Bugs
 _There are no bugs, just features! :P_

 Please use the github ticketing system. In this way other users might benefit from tips ans solutions from other users.
