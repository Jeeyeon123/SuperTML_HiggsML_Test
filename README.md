# SuperTML for HiggsML

Attempting to reproduce the results of [SuperTML - Sun et al., 2019](https://arxiv.org/abs/1903.06246) for the [Higgs ML Kaggle Challenge](https://www.kaggle.com/c/higgs-boson)

## Process

### Pre-processing and image creation 

#### As text ([image_formatter.ipynb](https://github.com/GilesStrong/SuperTML_HiggsML_Test/blob/master/notebooks/image_formatter.ipynb))

- Tabula data is pre-processed to:
  - Fix event orientation in phi, x, and z
  - Convert 3-momenta to Cartesian coordinates
- Events converted to 224x224 images by printing feature values as floats (3 d.p. precision) as text on black backgrounds

#### As pixels ([image_formatter_pixels.ipynb](https://github.com/GilesStrong/SuperTML_HiggsML_Test/blob/master/notebooks/image_formatter_pixels.ipynb))

- Tabula data is pre-processed to:
  - Fix event orientation in phi, x, and z
  - Convert 3-momenta to Cartesian coordinates
- Events converted to 224x224 images by colouring blocks of pixels according to feature values

### Classifier training, e.g. [2_resnet34_data-fix_Pixels-224.ipynb](https://github.com/GilesStrong/SuperTML_HiggsML_Test/blob/master/notebooks/2_resnet34_data-fix_Pixels-224.ipynb)

- CNN classifier constructed using Resnet34 pretrained on ImageNet as a backbone
- CNN is refined on training data in two stages:
  - 1st stage only trains final two dense layers
  - 2nd stage unfreezes and trains entire network (with discriminative learning rates)

### Inference, e.g. [2_resnet34_data-fix_Pixels-224.ipynb](https://github.com/GilesStrong/SuperTML_HiggsML_Test/blob/master/notebooks/2_resnet34_data-fix_Pixels-224.ipynb)

- Refined CNN applied to validation data
- Predictions converted to 1D prediction (zero = background, 1 = signal)
- AMS values calculated
- Cut on prediction chosen by taking the mean cut corresponding to the top 10% of AMS values
- Classifier applied to test data and AMS at chosen cut is evaluated

## Results

- First attempt:
  - equal fontisize
  - 224x224 images
  - Resnet34
  - Training does not account for event weights or balance classes
  - With no explicit regularisation, training-validation performance shows slight overtraining towards end, error rate bottoms out at ~17.5%
  - Maximum AMS on validation data is 2.91, AMS at chosen cut is 2.80
  - Public-private AMS on test data at chosen cut = 2.79-2.74
  - Appears to be better than random guessing, but currently worse than traditional binary classification and no where near the 3.979 reported

- Second attempt:
  - Moved to ariel 13 for font (text on previous images were a bit pixelated)
  - Shortened 2nd stage of training
  - Slightly improved performance:
    - min validation error-rate ~17%
    - Maximum AMS on validation data is 3.10, AMS at chosen cut is 3.02
    - Public-private AMS on test data at chosen cut = 2.78-2.83
  - Slight improvements with cleaner text labels, but still bad performance

- Third attempt
  - Encoded data as blocks of different pixel intensities by passing standardised & normalised data through a sigmoid and timesing by 255
  - Encoding was quicker and image file size was smaller (13 GB --> 9 GB)
  - Training slightly imporved: 16% error rate
  - Maximum AMS on validation data is 3.62, AMS at chosen cut is 3.44
  - Public-private AMS on test data at chosen cut = 3.29-3.32
  - Better encoding, but performance still worse than a 4-layer ReLU FCNN

- Fourth attempt
  - Same as third attempt, but using train-time data augmentation to adjust the contrast and brightness slightly during training
  - Training slightly worsened, but still around a 16% error rate
  - Similar validation performance: Maximum AMS on validation data is 3.61, AMS at chosen cut is 3.45
  - Similar test performance: Public-private AMS on test data at chosen cut = 3.24-3.36
  - Running test-time data augmentation worsened results considerably

## Requirements

- Python >= 3.6
- [fastai](https://github.com/fastai/fastai) > 1.0 - CNN training & inference
- [lumin](https://github.com/GilesStrong/lumin) == 0.2 - data processing, scoring
