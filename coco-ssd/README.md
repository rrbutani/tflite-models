## [COCO](http://cocodataset.org/#home) [SSD](https://towardsdatascience.com/understanding-ssd-multibox-real-time-object-detection-in-deep-learning-495ef744fab) Models

TFLite models for [the demo](https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd/demo) in [this project](https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd).

As the README in the project [notes](https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd#technical-details-for-advanced-users), the models in the original demo are converted from [models in Tensorfow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md#coco-trained-models) in the `tensorflow/models` repo.

The steps used are more or less the same as [the steps here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_on_mobile_tensorflowlite.md).

Currently, we've got the following models converted (full list [here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md#coco-trained-models)):
  - [ssd_mobilenet_v1_quantized_coco](http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_quantized_300x300_coco14_sync_2018_07_18.tar.gz)
  - [ssd_mobilenet_v2_quantized_coco](http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v2_quantized_300x300_coco_2019_01_03.tar.gz)
  - [ssdlite_mobilenet_v2_coco](http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz)
  - [ssd_resnet_50_fpn_coco](http://download.tensorflow.org/models/object_detection/ssd_resnet50_v1_fpn_shared_box_predictor_640x640_coco14_sync_2018_07_03.tar.gz)

Additionally:
 - [this](https://zenodo.org/record/3252084/files/mobilenet_v1_ssd_8bit_finetuned.tar.gz) finetuned MobileNet V1 model from Habana (from [this](https://github.com/mlperf/inference/tree/master/v0.5/classification_and_detection#mlperf-inference-benchmarks-for-image-classification-and-object-detection-tasks) list)

### To convert the models:
  - Clone the [`tensorflow/models` repo](https://github.com/tensorflow/models/).
  - Also go clone the [`tensorflow/tensorflow` repo](https://github.com/tensorflow/tensorflow/).
  - Install the things [here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md#dependencies) in a virtualenv (I used Python 3.7.4).
  - Make sure you've got a modern protoc installed (3.x; I used 3.5.1).
  - Run the [protobuf compilation step](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md#protobuf-compilation).
  - Follow the instructions [here](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/running_on_mobile_tensorflowlite.md):
    + Set the env vars to point to the untared model:
        * `CONFIG_FILE` <- pipeline.config
        * `CHECKPOINT_PATH` <- `<dir with checkpoints>/model.ckpt` (this is a prefix; no file with this name will exist)
        * `OUTPUT_DIR` <- whatever you want; will be made if it doesn't exist
    + Set your `PYTHONPATH` env var:
        * It should include:
            + the `models/research` dir (which, yes, you should be running the script from)
            + the `models/research/slim` dir
        * i.e.: `export PYTHONPATH="</path/to/models/repo>/research:</path/to/models/repo>/research/slim:${PYTHONPATH}"`
    + Run the command (be sure to use the python binary in your virtualenv):
        * also, pass the script into the python interpreter unlike the guide (so that you don't use the system python version to run it - pesky magic)
        * `python object_detection/export_tflite_ssd_graph.py  --pipeline_config_path="${CONFIG_FILE}" --trained_checkpoint_prefix="${CHECKPOINT_PATH}" --output_directory="${OUTPUT_DIR}" --add_postprocessing_op=true`
    + Finally, run TOCO (mostly) as instructed:
        * In the root of the `tensorflow/tensorflow` repo:
          ```bash
          bazel run tensorflow/lite/toco:toco -- \
            --input_file=$OUTPUT_DIR/tflite_graph.pb \
            --output_file=$OUTPUT_DIR/detect.tflite \
            --input_shapes=1,300,300,3 \
            --input_arrays=normalized_input_image_tensor \
            --output_arrays='TFLite_Detection_PostProcess','TFLite_Detection_PostProcess:1','TFLite_Detection_PostProcess:2','TFLite_Detection_PostProcess:3' \
            --inference_type=QUANTIZED_UINT8 \
            --mean_values=128 \
            --std_values=128 \
            --change_concat_input_ranges=false \
            --allow_custom_ops \
            --default_ranges_min=0 \
            --default_ranges_max=255
          ```
          - Note: the ResNet model's input tensor is 640x640; adjust the command above accordingly when converting it.
        * If everything worked, a working TFLite model should be in `$OUTPUT_DIR/detect.tflite`. 🤞
