#  <div align="center">Tutorial to train EfficientPS for Cityscapes dataset </div>

## Installation
You will need miniconda or conda to create virtualenv for this tutorial. If you haven't installed miniconda, here's a helpful link to install it https://www.youtube.com/watch?v=Avx_FYdFBcc

Install and setup dependencies for EfficientPS as follows:

```
git clone https://github.com/DeepSceneSeg/EfficientPS.git
cd EfficientPS
conda env create -n efficientPS_env --file=environment.yml
conda activate efficientPS_env
pip install -r requirements.txt
cd efficientNet
python setup.py develop
cd ..
python setup.py develop
```


## Download Cityscapes dataset using scripts 
Link to the tutorial (https://towardsdatascience.com/download-city-scapes-dataset-with-script-3061f87b20d7)

```
wget --keep-session-cookies --save-cookies=cookies.txt --post-data 'username=myusername&password=mypassword&submit=Login' https://www.cityscapes-dataset.com/login/
wget --load-cookies cookies.txt --content-disposition https://www.cityscapes-dataset.com/file-handling/?packageID=1
wget --load-cookies cookies.txt --content-disposition https://www.cityscapes-dataset.com/file-handling/?packageID=3
```
You will probably need packageID number 1 and number 3 which are gtFine_trainvaltest.zip and leftImg8bit_trainvaltest.zip for this tutorial


## Create symlink (symbolic link) of the dataset root to $EfficientPS/data
It is recommended to symlink the dataset root to $EfficientPS/data.

Change /home/user/ to whatever path you have on your local machine

```
ln -s /home/user/EfficientPS/datatest/gtFine /home/user/EfficientPS/data
```


## Convert Cityscapes dataset to correct efficientPS format
Change /home/user/ to whatever path you have on your local machine
```
python tools/convert_cityscapes.py /home/diho0521/EfficientPS/datatest ./data/cityscapes/
cd ..
git clone https://github.com/mcordts/cityscapesScripts.git
cd cityscapesScripts/cityscapesscripts/preparation
python createPanopticImgs.py --dataset-folder /home/user/EfficientPS/datatest/gtFine --output-folder ../../../EfficientPS/data/cityscapes --set-names val
```


## Training
a. Train with single GPU:

First modify the config file located at ./configs/efficientPS_singlegpu_sample.py. Go to line 208 where you can modify <b>total_epochs</b>. Default setting is 160 epochs.
Use --resume_from work_dirs/checkpoints/epoch_50.pth to continue train from pretrained weights e.g. epoch_50. If you start from stratch, remove this option.

```
cd ~/EfficientPS
python3 ./tools/train.py ./configs/efficientPS_singlegpu_sample.py --work_dir work_dirs/checkpoints --validate 
```
Note: Even though, the name suggests to use single gpu. I noticed the default setting will actually use 2 gpus. If you look at line 165 and 166 in the config file: imgs_per_gpu=2, workers_per_gpu=2,

b. Train with multiple gpu:

First modify the config file located at ./configs/efficientPS_multigpu_sample.py. Go to line 208 where you can modify <b>total_epochs</b>. Default setting is 160 epochs.
Use --resume_from work_dirs/checkpoints/epoch_50.pth to continue train from pretrained weights e.g. epoch_50. If you start from stratch, remove this option.
```
./tools/dist_train.sh ./configs/efficientPS_multigpu_sample.py ${GPU_NUM} --work_dir work_dirs/checkpoints --validate 
```
For example:
```
./tools/dist_train.sh ./configs/efficientPS_multigpu_sample.py 4 --work_dir work_dirs/checkpoints --validate --resume_from work_dirs/checkpoints/epoch_50.pth --validate 
```
This means CONFIG file=./configs/efficientPS_multigpu_sample.py, GPU_NUM=4 (use 4 gpus) and we use resume_from option to continue training. By using 4 gpus you can accelerate the training process 4 times faster.
## Evaluation Procedure on validation set

```
python3 tools/test.py ./configs/efficientPS_singlegpu_sample.py work_dirs/checkpoints/epoch_50.pth --eval panoptic
```
work_dirs/checkpoints/epoch_50.pth means use the pretrained weights from epoch_50.

Note: The author only provides single scale evaluation script.

## Performance
Using pretrained weights from epoch_50 on val set, achieve PQ(all) = 47.0%, PQ(things) = 41.4% and PQ(stuff) = 51.0%

<p>
<img width="850" src="50 epochs results.JPG">
</p>

## Inference:  

Change ./datatest/ to whatever path you have on your local machine in ./datatest/leftImg8bit/test

./datatest/leftImg8bit/test is where I store the test folder for inference. 
```
python3 tools/cityscapes_save_predictions.py ./configs/efficientPS_singlegpu_sample.py work_dirs/checkpoints/epoch_50.pth ./datatest/leftImg8bit/test ./infererence_singlegpu_result50
```
This means config file=./configs/efficientPS_singlegpu_sample.py, checkpoint = work_dirs/checkpoints/epoch_50.pth, input_folder_images=./datatest/leftImg8bit/test, ouput_folder=./infererence_singlegpu_result50 

Note: The folder containing images for inferencing must follow the structure required by EfficientPS where inside inside the folder containing images, there are the list of cities folder, inside each city folder there are images. For example in our folder_image ./datatest/leftImg8bit/test/, we have the following structure where berlin, bielefeld, bonn, etc. are the name of the cities the images are taken from.

<p>
<img width="300" src="tree structure of leftImg8bit.JPG">
</p>

## Inference result examples
I got 0.6 FPS for inference time using a single gpu. Below are some inference examples from the leftImg8bit/test set:

<p>
<img width="416" src="berlin_000000_000019_panoptic.png">
<img width="416" src="berlin_000001_000019_panoptic.png">
</p>
<p>
<img width="416" src="berlin_000002_000019_panoptic.png">
<img width="416" src="berlin_000003_000019_panoptic.png">
</p>


## Contact
Feel free to contact me if you have any issue. 

## Reference
Link to the original EfficientPS github https://github.com/DeepSceneSeg/EfficientPS





