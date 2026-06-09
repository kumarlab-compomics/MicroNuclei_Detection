

<div align="center">
<img src="./sample_images/example1.png" alt="sample" width="600" height="500">
</div>

## 🏠 Introduction

The **MicroNucML** repository provides a comprehensive end-to-end pipeline for the automatic detection and segmentation of micronuclei (MN) and their parent nuclei in microscopy images. By leveraging high quality ground truth, and modern deep learning architecture, this tool is designed to help researchers and practitioners in biomedical imaging, to quickly and accurately identify MN at-scale. MN can be used as a biomarker for chromosomal instability, and can be applied to study cancer, therapeutics, and environmental carcinogens. 

### Key Features

- A flexible command-line tool interface, enabling the processing of large image sets via a simple `python image_process.py --src ... --dst ...` invocation, with configurable modes (nuclei only, micronuclei only, or both), confidence thresholds, and parent-assignment strategies.
  
- A seamless integration of segmentation of nuclei and micronuclei.

- Support for multiple modern back-ends and pretrained checkpoints (e.g., ResNet50, Swin, ResNet101) to enable rapid setup and experimentation. 

- Tool outputs includes support for segmentation-based datasets (via JSON annotations, RLE encoding, bounding boxes, and masks), classification scores, and parent nuclei-MN assignment. Output formats clearly defined for downstream analytics and data pipelines. 

- The tool is designed to have a modular code structure including, data-loading, training, inference (for both segmentation and classification). Published notebook have been released to provide examples in statistical evaluation, plotting, and running the tool.

## 🔥 News

- [2026/01] [MN Ground truth dataset]( https://data.mendeley.com/datasets/hrjn4dy6z9/1) is released.
- [2025/09] [MicroNucML preprint]( https://www.biorxiv.org/content/10.1101/2025.09.20.677550v1) is released.
- [2025/06] MicroNuclei Detection v1.0 released.
  
## 📦 Installation
This tool need to be installed before use. All the requirements are in `requirements.txt` (for Compute Canada users, see `requirements_ComputeCan.txt`). Please install pytorch and torchvision dependencies. 
```
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

You can install this tool on a GPU machine using:

```
git clone https://github.com/kew6688/MicroNuclei_Detection.git && cd MicroNuclei_Detection/src
pip install -r requirements.txt
pip install -e .
```

In order to fully utilize the pipeline (ie. nuclei segmentation), we integrate using SAM2.

## 📚 Model Checkpoints

First, we need to download a model checkpoint. All the model checkpoints can be downloaded by running:

```
cd MicroNuclei_Detection/src/checkpoints && ./download_ckpts.sh
```

The pre-trained models can be download from huggingface:
- [resnet50](https://huggingface.co/ccglab22/MaskRCNN-resnet50FPN)
- [swin](https://huggingface.co/ccglab22/MaskRCNN-swinFPN)
- [resnet101](https://huggingface.co/ccglab22/MaskRCNN-resnet101)
  
After downloading the model, the usage of the end-to-end pipeline is described below.

Download SAM2 checkpoint by running:
```
cd MicroNuclei_Detection/external/sam2/checkpoints && \
./download_ckpts.sh && \
cd ..
```

## 🧩 Usage:
### **Using the tool as a library.** 
This allows for:
 - predicting MN counts
 - creating MN masks
 - the option to conduct nuclei segmentation, and subsequent nuclei-MN integration
 - customizable parameters for MN segmentation confidence and filtering for apoptosis-like structures

Please see this example in the [UpdatedTutorial.ipynb](./src/notebooks/UpdatedTutorial.ipynb). This tutorial can also be run using GPU resources provided by [Google Colab](https://colab.research.google.com/github/kumarlab-compomics/MicroNuclei_Detection/blob/main/src/notebooks/UpdatedTutorial.ipynb)

### **Run the main python script.** 
Automated pipeline to process images. 

```
Example Usage:
      >>> python MicroNuclei_Detection/src/compute_scripts/image_process.py --src ./sample_images --dst test.json 
```
**Input**: The model expects 8-bit RGB images (tif, png, jpg) text-less images. The training data is at 20x magnification, and at 0.625 µm/pixel resolution.

**Output**: Please see [output](#output-format) section

### Parameters and Arguments
| Parameter          | Short Form | Required | Default    | Type         | Description                                                                                       |
|--------------------|------------|----------|------------|--------------|---------------------------------------------------------------------------------------------------|
| `--src`            | `-s`       | Yes      | N/A        | String       | Path to image.                                                                                 |
| `--dst`            | `-d`       | Yes      | N/A        | String       | Path to output.                                                                                |
| `--mode`           | `-mod`     | No      | "ALL"        | String       | Segmentation mode and output. ALL for both nuclei and MN segmentation, NUC for only nuclei segmentation, MN for only micronuclei segmentation. Options: `["MN", "NUC", "ALL"]`                                                  |
| `--conf`           | `-c`       | No       | 0.7        | Float        | Minimum confidence threshold for MN detection, Options: value between `[0 to 1]`                        |
| `--parent`         | `-p`       | No       | "edge"        | String        | Parent assign method, when assigning parent nuclei to MN. MN centroids are identified, then nuclei's closest centroid or edge is used, Options: `["center", "edge"]`   |
| `--apop`         | `-apop`       | No       | True        | Bool        | Turn ON/OFF the apoptosis check function, Options: `["True", "False"]`   |
| `--apop_cnt`         | `-apop_cnt`       | No       | 5        | Integer        | The minimum number of MN within a DBSCAN cluster to be considered potentially apoptotic  |
| `--mask`         | `-mask`       | No       | N/A       | String       | The folder location of input mask for nuclei segmentation, the masks in the folder should have the same name matched to the images. The shape expected is [n, w, h], n is number of nuclei, w,h is image shape |



## Project Structure:

```bash
├ src
├── checkpoints           # Download pretrained weight here
├── mn_classification     # Micronuclei classification training and inference files
│   ├── utils             # Utils code (generate dataset, generate tiles contained mn)
│   ├── data_load.py      # Datasets loader (PyTorch)
│   ├── test.py           # Inference code for experiment statistics and plots
│   ├── train.py          # Trainer functions to train networks
│   └── main.py           # Run file to start an experiment
├── mn_segmentation       # Micronuclei Segmentation training and inference files
│   ├── datasets          # Datasets (PyTorch)
│   ├── lib               # Application class for using the model
│   ├── models            # Networks architecture
│   ├── train             # Trainer functions to train networks
│   ├── tests             # Evaluation code
│   └── run.py            # Run file to start an experiment
└── notebooks             # Inference code for experiment statistics and plots
```

## Dependency

https://stackoverflow.com/questions/714063/importing-modules-from-parent-folder

Build project from root dir. Import by prepending MN.

## Dataset 
The following is the overview of the dataset. The dataset can be download [from Mendeley](https://data.mendeley.com/datasets/hrjn4dy6z9/1). By downloading the dataset you agree that you have read and accept the licence.

```
H2B-GFP/
├── test_image          # Held-out tiles (.pngs) used for testing
├── test_mask           # Mask of MN from held-out tiles (.npy) used for testing
├── train_image         # Tiles (.pngs) used for training
├── train_mask          # Mask of MN from training tiles (.npy)

H2B-mCherry/
├── grey_image          # Grey-scaled H2B-mCherry tiles (.pngs) used when labelling
├── grey_mask           # Masks of MN from labelled grey-scaled H2B-mCherry tiles (.npy)
├── red_image           # Pseudo-red H2B-mCherry tiles (.pngs) used when labelling
├── red_mask            # Masks of MN from labelled pseudo-red H2B-mCherry tiles (.npy)
```

We save masks in a .npy file, at a tile-level, for each object.

## Output format
The generated output (.json), can include predictions from nuclei and MN segmentations. They are stored in the following format:

```
[image_info], size n == number of images:

[
    {
        "image": image_name,    # str
        "nuclei": nuc_info,
        "micronuclei": mn_info
    }
]

For each info:

{
      "coord": [[x1, y1],...],   `# list of center coordinates
      "area": [x,...],            # list of mask area
      "bbox": [...],              # list of bounding box, 
                                    for MN bbox: (xmin, ymin, xmax, ymax)
                                    for nuclei bbox: (x, y, w, h) 
      "score": [x,...],           # list of prediction scores for each object
      "mask": [[...],...]         # list of rle encoding list for each object

      "parent": [id,...]          # assigned parent nuclei, mn only
}
```
Note: RLE encoding and decoding functions can be find in 
```
from mn_segmentation.lib.image_encode import mask2rle, rle_to_mask
rle = mask2rle(mask)
mask = rle_to_mask(rle,original_height,original_width)
```




