# package_segmentation
This is a package segmentation ROS package.

# Annotation and training

## 1. Preparing images
### Acceptable image format
- 1024pix X 1024pix  
- (3channels, 8bit, grayscale) or (1 channel, 8bit, grayscale)
- bmp or png  
- If there are any boxes outside the pallet, to exclude those boxes, you should paint the area outside pallet on the images  (with pixel value equals 0). It is because the boxes outside the pallet likely will be left and staying in multiple images. Consequently they will be learned repeatably and the learned network will be biased.

## 2. Location of annotation data files, image files and via.html
Locate image files for training and validation like the following example. Other files will be created in the following steps.  
Example:
```
Rapyuta_training
	├── annotations
	│   └── 20191003
	│       ├── train
	│       │   ├── via_region_data_20190911_train_data_rakuten_etc.json
	│       │   └── via_region_data_4.0_rakuten.json
	│       └── val
	│           └── via_region_data_4.0_yodobashi.json
	│
	├── 20190911_train_data_rakuten_etc
	│   ├── left0001.png
	│   ├── left0002.png
	│   ├── left0003.png
	│   └── left0004.png
	│
	├── JP_realistic_site_data
	│   ├── 4.0_rakuten
	│   │   ├── 1_projector_off.png
	│   │   ├── 2_projector_off.png
	│   │   ├── 3_projector_off.png
	│   │   ├── 4_projector_off.png
	│   │   ├── 5_projector_off.png
	│   │   ├── 6_projector_off.png
	│   │   └── 7_projector_off.png
	│   └── 4.0_yodobashi
	│       ├── 1_projector_off.png
	│       ├── 2_projector_off.png
	│       ├── 3_projector_off.png
	│       ├── 4_projector_off.png
	│       └── 5_projector_off.png
	└── via.html

```

### 'annotation_dir' and 'image_dir' directory
The 'annotation_dir' directory is /Rapyuta_training/annotations/20191003/ in the above location of files.  
The argument on training execution should be set as '--annotation_dir=/Rapyuta_training/annotations/20191003/'.  
The 'annotation_dir' directory has:
- 'train' directory
- 'val' directory

The 'image_dir' directory is /Rapyuta_training/ in the above location of files.  
The argument on training execution should be set as '--image_dir=/Rapyuta_training/'.  
The 'image_dir' directory has:
- image files or directory including image files
- via.html

### Program behavior
First, according to '--annotation_dir=', the program finds 'train' and 'val' directories, which include annotation data file 'via_region_data\*.json'.  
Second, Images for training are found according to '--image_dir=' and 'filename' in 'via_region_data\*.json'. 'filename' can include relative path from 'image_dir'.  

## 3. Annotation with VIA
### 1. Create annotation data files
1. Prepare the annotation software VIA(via-2.0.8.zip).  
http://www.robots.ox.ac.uk/~vgg/software/via/
1. Put 'via.html' in 'image_dir' directory.
1. Make file list of the image files. Relative path from 'image_dir' directory. Text file. This file name and location is arbitrary.
```
[20190911_train_data_rakuten_etc_file_list.txt]
20190911_train_data_rakuten_etc/left0001.png
20190911_train_data_rakuten_etc/left0002.png
20190911_train_data_rakuten_etc/left0003.png
20190911_train_data_rakuten_etc/left0004.png
```
```
[4.0_rakuten_file_list.txt]
JP_realistic_site_data/4.0_rakuten/1_projector_off.png
JP_realistic_site_data/4.0_rakuten/2_projector_off.png
JP_realistic_site_data/4.0_rakuten/3_projector_off.png
JP_realistic_site_data/4.0_rakuten/4_projector_off.png
JP_realistic_site_data/4.0_rakuten/5_projector_off.png
JP_realistic_site_data/4.0_rakuten/6_projector_off.png
JP_realistic_site_data/4.0_rakuten/7_projector_off.png
```
```
[4.0_yodobashi_file_list.txt]
JP_realistic_site_data/4.0_yodobashi/1_projector_off.png
JP_realistic_site_data/4.0_yodobashi/2_projector_off.png
JP_realistic_site_data/4.0_yodobashi/3_projector_off.png
JP_realistic_site_data/4.0_yodobashi/4_projector_off.png
JP_realistic_site_data/4.0_yodobashi/5_projector_off.png
```
1. Open 'via.html'.
1. In VIA, select menu "Project"->"Add url or path from text file".
1. Designate one of the above 3 file lists.  
'via.html' will read images based on the directory where 'via.html' is located.
1. Annotate with "Polygon region shape".
1. To export annotation data file, select menu "Annotation"->"Export Annotations (as json)". Save as the following names.
1. Repeat the above 3 operations to make 3 annotation data files, like
- via_region_data_20190911_train_data_rakuten_etc.json
- via_region_data_4.0_rakuten.json
- via_region_data_4.0_yodobashi.json  
  
  File name needs to follow the format: 'via_region_data\*.json'  

### 2. Classify the annotation data files for training and validation
Classify the annotation data files created in the above operation for training and validation.  
Here they will be treated as  
- For training  
via_region_data_20190911_train_data_rakuten_etc.json  
via_region_data_4.0_rakuten.json  
- For validation  
via_region_data_4.0_yodobashi.json  
  
There is no difference between trainig json and validation json as to the way to create them.  

## 4. Training
### Training execution
1. Be sure that you did locate the image files for training and validation in 'image_dir' directory.
1. Locate 'via_region_data\*.json' files for training in 'train' directory in the 'annotation_dir' directory.  
Locate 'via_region_data\*.json' files for validation in 'val' directory in the 'annotation_dir' directory.  
Multiple files can be included in 'train' and 'val' directory respectively.
1. Locate 'mask_rcnn_coco.h5' in '/package_segmentation/src/model/mask_rcnn_coco.h5' if you want to start with non-trained-for-box weights. 'mask_rcnn_coco.h5' is uploaded in Google Drive /Colab/data/weights/initial_weight/mask_rcnn_coco.h5  
Or locate trained-for-box weights like 'mask_rcnn_package_0040.h5' in '/package_segmentation/src/model/mask_rcnn_package_0040.h5' if you want to continue learning.
1. In your console, change the directory to '/package_segmentation/src/'
1. Execute package_segmentation.py with the following arguments like,
```
python package_segmentation.py train --annotation_dir=/Rapyuta_training/annotations/20191003/ --image_dir=/Rapyuta_training/ --weights=coco
```
 1. Designate 'train'.
 1. Designate the annotation_dir directory using the argument.  
 This should be absolute path.  
'--annotation_dir=/Rapyuta_training/annotations/20191003/'  
 1. Designate 'image_dir' directory using the argument.  
 This should be absolute path.  
'--image_dir=/Rapyuta_training/'  
 1. Designate the weights file to start learning using the argument.  
 - To start with non-trained-for-box weights, i.e.'mask_rcnn_coco.h5',  
'mask_rcnn_coco.h5' should be located in '/package_segmentation/src/model/mask_rcnn_coco.h5'  
'mask_rcnn_coco.h5' is uploaded in Google Drive /Colab/data/weights/initial_weight/mask_rcnn_coco.h5  
'--weights=coco'  
 - To start with trained-for-box weights like 'mask_rcnn_package_0040.h5'  
An absolute path:  
'--weights=/package_segmentation/src/logs/package20190919T2210/mask_rcnn_package_0040.h5'  
A relative path from 'ROOT_DIR/model/':  
'--weights=../logs/package20190919T2210/mask_rcnn_package_0040.h5'  
'--weights=mask_rcnn_package_0040.h5'  
1. The trained weights will be exported in ROOT_DIR/logs/, like ROOT_DIR/logs/package20190919T2210/mask_rcnn_package_0040.h5.  
They will be exported at every epoch completing.  
You can designate the exporing directory with absolute path as:  
'--logs=/package_segmentation/src/something_logs/'
1. There is no special teminating routines when program exiting, so you can abort the program running at any time.  

### Training parameters
Training parameters are set in the source code.  
See the followings.
```
package_segmentation.py
class PackageConfig(Config):
```
```
config.py
class Config(object):
```

# Appendix
## VIA annotation file format
"via_region_data\*.json"  
  
1st line:  
Name of data item. It is not actually file name. It is arbitrary if it is unique.  
  
2nd line:  
File name.  
  
If you want to change the path of image file, please change the 2nd line. It is OK to change the 1st line also.  
  
```
{
	"20190911_train_data_rakuten_etc/left0001.png-1": {
		"filename": "20190911_train_data_rakuten_etc/left0001.png",
		"size": -1,
		"regions": [
```

# How to use the interface functions

## To detect box
The following two function can be utilized.

def create_model_to_detect_package_with_2D_image(weights_path, logs_path=DEFAULT_LOGS_DIR):  
def detect_package_with_2D_image(model, image):  

1. Put the weights file in this directory.  
/package_segmentation/src/model/mask_rcnn_package_1300.h5  

2. Execute 'create_model_to_detect_package_with_2D_image'.  
  This process takes time. The process should be keep alive to keep model with weights in main memory.  

3. Execute 'detect_package_with_2D_image'.  
  - 1024pix X 1024pix  
  - (3channels, 8bit, grayscale) or (1 channel, 8bit, grayscale)  
  The above image data is acceptable.  
  With image of other sizes, the program will be aborted.  

See codes in '__main__' part of package_segmentation.py  

