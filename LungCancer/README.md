
# Preliminary comment: On the NYUm HPC cluster, 3 modules are needed. The commands mentioned below must be run through qsub scripts and not on the head node! Module needed are:
module load cuda/8.0
module load python/3.5.3
module load bazel/0.4.4


# 1 - Prepare the images
# See from https://github.com/tensorflow/models/blob/master/inception/README.md for specific format needed by inception


# for the whole training set, the following code was used to convert JPEG to TFRecord:
python build_image_data.py --directory='jpeg_main_directory' --output_directory='outputfolder' --train_shards=1024 --validation_shards=128 --num_threads=4

The jpeg must not be directly inside 'jpeg_main_directory' must in subfolders with names corresponding to the labels
jpeg_main_directory/TCGA-LUAD
jpeg_main_directory/TCGA-LUSC


# The same was done for the test set:
python build_image_data.py --directory=/ifs/home/coudrn01/NN/Lung/Test_All512pxTiled/tmp_4_2Types/ --output_directory=/ifs/home/coudrn01/NN/Lung/Test_All512pxTiled/tmp_4_2Types/ --train_shards=20 --validation_shards=20 --num_threads=4 

# This code was adapted from https://github.com/awslabs/deeplearning-benchmark/blob/master/tensorflow/inception/inception/data/build_image_data.py




# 2 - Re-training from scratch

# Build the model. Note that we need to make sure the TensorFlow is ready to
# use before this as this command will not build TensorFlow.
cd 01_training/inception
bazel build inception/imagenet_train

# run it for all the training images:
bazel-bin/inception/imagenet_train --num_gpus=1 --batch_size=30 --train_dir='output_directory' --data_dir='TFRecord_images_directory'

botteneck, graph, variables... are saved in the output_directory 



# Briefly, one can evaluate the model by running a test on the validation set (must be started on a different node from the one used for training):
## to prepare the run:
bazel build inception/imagenet_eval
## to actually run it:
bazel-bin/inception/imagenet_eval --checkpoint_dir='checkpoint_dir' --eval_dir='output_dir' --run_once --data_dir='validation_TFRecord_images'

#(The precision @ 1  measures how often the highest scoring prediction from the model matched the  label)=
# (=Much like the training script, imagenet_eval.py also exports summaries that may be visualized in TensorBoard:

tensorboard --logdir='checkpoint_dir'

