- #  Setup simpledet

- ## 1. setup simpledet by docker

- ### 1.1 System Requirements

    ubuntu16.04

    python >=3.5

- ### 1.2 Download docker images

    The version of docker image: ubuntu16.04， cuda9.0， cudnn7， python3.

    <https://gitlab.com/nvidia/cuda/blob/ubuntu16.04/9.0/devel/cudnn7/Dockerfile>

- ### 1.3 Run docker container

    nvidia-docker run -v $HOST-SIMPLEDET-DIR:$CONTAINER-WORKDIR -it nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04 bash


- ### 1.4 Install dependency(in docker container) 

```
# Install dependency
sudo apt-get update
sudo apt-get install -y build-essential git
sudo apt-get install -y libopenblas-dev
```
- ### 1.5 Download tools, include simpledet, pycocotools and mxnext

```
git clone <https://github.com/TuSimple/simpledet.git>
cd /path/to/simpledet
make

# Install a patched cocotools for python3
git clone <https://github.com/RogerChern/cocoapi>
cd cocoapi/PythonAPI
python3 setup.py install

# setup mxnext, a wrapper of mxnet symbolic API
cd /path/to/simpledet
git clone <https://github.com/RogerChern/mxnext>
```

- ### 1.6 Install mxnet
```
# Specify simpledet directory
export SIMPLEDET_DIR=/path/to/simpledet
export COCOAPI_DIR=/path/to/cocoapi

git clone <https://github.com/apache/incubator-mxnet> mxnet
cd mxnet
git checkout 1.3.1
git submodule init
git submodule update
echo "USE_OPENCV = 0" >> ./config.mk
echo "USE_BLAS = openblas" >> ./config.mk
echo "USE_CUDA = 1" >> ./config.mk
echo "USE_CUDA_PATH = /usr/local/cuda" >> ./config.mk
echo "USE_CUDNN = 1" >> ./config.mk
echo "USE_NCCL = 1" >> ./config.mk
echo "USE_DIST_KVSTORE = 1" >> ./config.mk
cp -r $SIMPLEDET_DIR/operator_cxx/* src/operator/
mkdir -p src/coco_api
cp -r $COCOAPI_DIR/common src/coco_api/
make -j
cd python
python3 setup.py install
```

- ## 2. test on MSCOCO dataset

- 2.1 Download the detection model

    Make new directory below the path of simpledet, and named 'experiments', place the model you download in this directory. You can download the models provided by the author via <https://github.com/TuSimple/simpledet/tree/master/models/tridentnet> .
```
experiments/
    tridentnet_r101v2c4_c5_1x/
        checkpoint-0006.params
        checkpoint-symbol.json
        log.txt
        coco_minival2014_result.json
```
- 2.2 Construct coco roidb
Place the coco dataset as show below:

```
data/
    coco/
        annotations/
            instances_train2014.json
            instances_valminusminival2014.json
            instances_minival2014.json
            image_info_testdev2017.json
        images/
            train2014
            val2014
            test2017
```

- 2.3 Generate coco roidb
```
python3 utils/generate_roidb.py --dataset coco --dataset-split train2014
python3 utils/generate_roidb.py --dataset coco --dataset-split valminusminival2014
python3 utils/generate_roidb.py --dataset coco --dataset-split minival2014
python3 utils/generate_roidb.py --dataset coco --dataset-split test-dev2017
```

- 2.4 Test
```
python3 detection_test.py --config config/detection_config.py
```

- ## 3. Detect on single image

Reference <https://github.com/danpe1327/simpledet/blob/master/detect_image.py>