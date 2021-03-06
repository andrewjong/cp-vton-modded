# Fork Description
This fork is our modifications to the original cp-vton code for the [2020 VUHCS video virtual try-on challenge](https://vuhcs.github.io/). The main difference is the Dataset classes that load the competition's VVT and MPV datasets.

To get results of the *original baseline*, do not use this repository. Instead use the original [sergeywong/cp-vton](https://github.com/sergeywong/cp-vton).


## Toward Characteristic-Preserving Image-based Virtual Try-On Network

Reimplemented code for eccv2018 paper 'Toward Characteristic-Preserving Image-based Virtual Try-On Network'. 

The results may have some differences with those of the original code.

This code is tested with pytorch=1.4.0, torchvision=0.5.0 and CUDA=10.1. Install with `conda env create -f environmental.yml`.

## Data preprocessing
<details>
<summary>Manually generate dataset</summary>
<br>
We convert the original data [VITON](https://github.com/xthan/VITON) into different directories for easily use. 

Run the matlab code ```convert_data.m ``` under the original data root ```VITON/data```, and get the new format.

We use the json format for pose info as generated by [OpenPose](https://github.com/CMU-Perceptual-Computing-Lab/openpose).

Move these directories into our own dataroot ```data```.
</details>



You can get the processed data at [GoogleDrive](https://drive.google.com/open?id=1MxCUvKxejnwWnoZ-KoCyMCXo3TLhRuTo) or by running:

```bash
python data_download.py
```

## Geometric Matching Module

### training
We just use L1 loss for criterion in this code. 

TV norm constraints for the offsets will make GMM more robust.

For Vera:

First checkout the `viton_vvt_mpv` branch.
```bash
git fetch
git checkout viton_vvt_mpv
```
Then train
```bash
echo "Train CP-VTON GMM"; \
python train.py \
--name train_gmm_cp-vvt-mpv_$(date +"%Y-%m-%d_%H-%M-%S") \
--stage GMM \
--shuffle \
--save_count 5000 \
--dataset cp_vvt_mpv \
--dataroot ./data `# or /data_hdd/cp-vton/viton_processed` \
--vvt_dataroot /data_hdd/vvt_competition \
--mpv_dataroot /data_hdd/mpv_competition  \
--workers 32 \
--gpu_ids 0,1,2,3,4,5,6,7 \
--batch_size 128
```
You can see the results in tensorboard, as show below. 
```bash
tensorboard --logdir tensorboard  # recommended to do this in a tmux window
```
We can port forward the training like this
```bash
echo "tensorboard connection"; ssh -N -L localhost:6006:localhost:6006 username@10.52.0.34
```
<div align="center">
  <img src="result/gmm_train_example.png" width="576px" />
    <p>Example of GMM train. The center image is the warped cloth.</p>
</div>

### eval

Choose the different source data for eval with the option ```--datamode```.

An example training command is
```bash
DATAMODE="train" `# choose train or test` \
python test.py \
--name gmm_traintest_new \
--stage GMM \
--workers 4 \
--datamode "$DATAMODE" \
--dataset cp_vvt_mpv \
--data_list "$DATAMODE"_pairs.txt \
--vvt_dataroot /data_hdd/vvt_competition \
--mpv_dataroot /data_hdd/mpv_competition \
--checkpoint checkpoints/gmm_train_new/gmm_final.pth
```

You can see the results in tensorboard, as show below.

<div align="center">
  <img src="result/gmm_test_example.png" width="576px" />
    <p>Example of GMM test. The center image is the warped cloth.</p>
</div>

## Try-On Module
### training
Before the trainning, you should generate warp-mask & warp-cloth, using the test process of GMM with `--datamode train`. 
**Then move these files or make symlinks under the directory `data/train`.**
```bash
# Link CPDataset
ln -s $(readlink -f ./results/"$CHECKPOINT"/CPDataset/warp-cloth) ./data/train/warp-cloth
ln -s $(readlink -f ./results/"$CHECKPOINT"/CPDataset/warp-mask) ./data/train/warp-mask

# Link VVTDataset
ln -s $(readlink -f ./results/"$CHECKPOINT"/VVTDataset/warp-cloth) /data_hdd/vvt_competition

# Link MPVDataset
ln -s $(readlink -f ./results/"$CHECKPOINT"/MPVDataset/warp-cloth) /data_hdd/mpv_competition
```

An example training command is

```bash
echo "Train CP-VTON TOM"; \
python train.py \
--name train_tom_cp-vvt-mpv_$(date +"%Y-%m-%d_%H-%M-%S") \
--stage TOM \
--shuffle \
--save_count 5000 \
--dataset cp_vvt_mpv \
--dataroot ./data `# or /data_hdd/cp-vton/viton_processed` \
--vvt_dataroot /data_hdd/vvt_competition \
--mpv_dataroot /data_hdd/mpv_competition  \
--workers 32 \
--gpu_ids 0,1,2,3,4,5,6,7 \
--batch_size 128
```
You can see the results in tensorboard, as show below.

<div align="center">
  <img src="result/tom_train_example.png" width="576px" />
    <p>Example of TOM train. The center image in the last row is the synthesized image.</p>
</div>


### eval
An example training command is

```
python test.py --name tom_test_new --stage TOM --workers 4 --datamode test --data_list test_pairs.txt --checkpoint checkpoints/tom_train_new/tom_final.pth
```

You can see the results in tensorboard, as show below.

<div align="center">
  <img src="result/tom_test_example.png" width="576px" />
    <p>Example of TOM test. The center image in the last row is the synthesized image.</p>
</div>


## Citation
If this code helps your research, please cite our paper:

	@inproceedings{wang2018toward,
		title={Toward Characteristic-Preserving Image-based Virtual Try-On Network},
		author={Wang, Bochao and Zheng, Huabin and Liang, Xiaodan and Chen, Yimin and Lin, Liang},
		booktitle={Proceedings of the European Conference on Computer Vision (ECCV)},
		pages={589--604},
		year={2018}
	}


