<div id="top" align="center">

# OG-HFYOLO :Orientation Gradient Guidance and Heterogeneous Feature Fusion For Deformation Table Cell Instance Segmentation
  
  Long Liu, Cihui Yang* </br>
  
  [![arXiv](https://img.shields.io/badge/arXiv-2504.20682-b31b1b.svg)](https://arxiv.org/abs/2504.20682)

</div>

## Datasets
### Data generator
The data generator mentioned in the paper is specifically implemented in [./dataprocess/data_gen.py](./dataprocess/data_gen.py), and detailed usage instructions are provided at the beginning of the file.

Additionally, the [./dataprocess](./dataprocess) folder contains scripts that can convert various dataset formats, including ([yolo2coco](./dataprocess/yolo2coco.py), [yolo2labelme](./dataprocess/yolo2labelme.py), [labelme2yolo](./dataprocess/labelme2yolo.py), [labelme2coco](./dataprocess/labelme2coco.py)), with specific usage instructions detailed at the beginning of each file.

### Datasets
Deformed Wire Table for Small (DWTAL-s) with 8,765 simpler tables mainly derived from TAL-OCR, and Deformed Wire Table for Large (DWTAL-l) containing 19,520 complex tables mainly expanded from WTW. Both datasets follow identical split strategy. To ensure uniform distribution of deformation types across training and test sets, 80\% of each dataset is randomly allocated for training and 20\% for testing. Final counts reach 7,012 training and 1,753 test images in DWTAL-s, versus 15,616 training and 3,904 test images in DWTAL-l.

#### *🔥YOLO version:*

We store the YOLO dataset format in Google Drive, you can download: 

*[DWTAL-s](https://drive.google.com/file/d/1i4meTuVevdtEUd7wde59Y7KzR27Dj9QF)* 

*[DWTAL-l](https://drive.google.com/file/d/1wJiRt2u7sY9uqZxJtSJiWy_Zhu87yQOU)* 
 
the YOLO dataset format as follows:
```
dataset
├──images
│   ├──train
│   │   ├── imaget.png|jpg
│   │   └── *.png|jpg
│   └──val
│       ├── imagev.png|jpg
│       └── *.png|jpg
└──labels
    ├──train
    │   ├── imaget.txt
    │   └── *.txt
    └──val
        ├── imagev.txt
        └── *.txt
```

#### *🔥COCO version:*
We store the COCO dataset format in Huggingface, you can download:

*[DWTAL-s](https://huggingface.co/datasets/justliulong/DWTAL/resolve/main/DTAL-s.coco.zip)* 

*[DWTAL-l](https://huggingface.co/datasets/justliulong/DWTAL/resolve/main/DTAL-l.coco.zip)* 

 *or* You can download the COCO versions of the dataset at one time through the following command

 ```python
from datasets import load_dataset
dataset = load_dataset("justliulong/DWTAL")
 ```

the COCO dataset format as follows:
 ```
 output
├── annotations
│   ├── train.json
│   └──  val.json
│ 
├── train
│   ├── imaget.png
│   └── *.png
└── val
    ├── imagev.png
    └── *.png
 ```

Additionally, we have provided the labelme format (without images) for the corresponding dataset, which can be found under *[DWTAL-s.json.zip](https://drive.google.com/file/d/1_FJTwJWV3AwnaLHhpwGq9K4t-ywCfmV-)* and *[DWTAL-L.json.zip](https://drive.google.com/file/d/10H2oc_kQIlHyHXim4xSSrwDPivOciw-8)*. You can place the images and JSON labels in the same folder and use labelme to open the images and view the annotations.

Finally, we have also provided the dataset with logical coordinates corresponding to DWTAL dataset, to advance the development of deformable table structure recognition:

The dataset with logical coordinate annotations is published in Releases (without images. The images need to be downloaded from the COCO version or YOLO version of the dataset). The logical coordinate annotations of all the images in the DWTAL-l dataset are stored in *[DWTAL-l.logical.zip](https://github.com/justliulong/OGHFYOLO/releases/download/v1.0.0/DTAL-l.logical.zip)*, containing "logical_coordinates" for each cell: 
```json
{
  "start_row": a,
  "end_row": b, 
  "start_col": c, 
  "end_col": d
}
```
- logical_coordinates uses 1-base, that is, the minimum of start_row and start_col starts from 1 instead of 0.

and the entire html sequence of the table. The logical coordinate annotations of approximately 80% of the images in the DWTAL-s dataset are stored in *[DWTAL-s.logical.zip](https://github.com/justliulong/OGHFYOLO/releases/download/v1.0.0/DTAL-s.logical.zip)*, including the html sequence of each table.



## Model
The model structure mentioned in the paper is specifically implemented in [./cfg/models/segment/og-hfyolo.yaml](./cfg/models/segment/og-hfyolo.yaml).


### Environments
Install requirements.txt in a Python>=3.8.0 environment, including PyTorch>=1.8.
```
conda create -n py38 python=3.8
conda activate py38
pip install -r requirements.txt
```
or
```
 conda env create -f environment.yaml
```

### Training CLI
🔥🔥**Before training, you need to prepare the dataset in YOLO format in advance and place it in the specified folder, such as '[./datasets](./datasets)'. Modify the dataset YAML file under '[cfg/datas](./cfg/data)'**, and execute the command in the project root directory:
```
python segment/train.py
```
By default, the script will use the DWTAL-s dataset for training and the OG-HFYOLO model for training.

### Valing CLI

```
python segment/val.py
```
🔥🔥**If the mask_nms post-processing mentioned needs to be used, it can be set by adding the parameter '--m_nms'**
```
python segment/val.py --m_nms ...<other param>
```

### Predict CLI

```
python segment/predict.py --weights /path/to/weights.pt --source /path/to/test_image/ < --instance > < --m_nms >
```
Optional parameters：
- `instance` is used to display the result as the instance segmentation result on the image, and try to assign different mask colors to adjacent instances as much as possible

- `m_nms` is used for post-processing with mask_nms

If you want to obtain masked images with higher resolution, you can add the '--retina-masks' parameter. However, in cases with a large number of instances, it will require more computing resources.

> Here, a demonstration weight is provided in [best.pt](https://github.com/justliulong/OGHFYOLO/releases/download/v1.0.2/best.pt). You can download the weights and place them in the `./sample_pt/` directory of the project, and run it directly with the following command for prediction:

```
python segment/predict.py --weights sample_pt/best.pt --source ./sample_pt/test_image/ --instance --m_nms
```
The results will be stored by default in the './runs/predict-seg/exp 'folder

## License
OG-HFYOLO is released under the GNU Affero General Public License v3.0 (AGPL-3.0). Please see the [LICENSE](./LICENSE) file for more information.

## Copyright Notice
Many utility codes of our project base on the codes of [Ultralytics YOLO](https://github.com/ultralytics/yolov5)