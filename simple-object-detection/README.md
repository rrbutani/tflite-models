## Simple Object Detection Model

A TFLite model for [this demo](https://github.com/tensorflow/tfjs-examples/tree/master/simple-object-detection).

In the original demo, the model is either generated server side by a [NodeJS script](https://github.com/tensorflow/tfjs-examples/blob/fb430042376431fba804a2495891100f6e2205d6/simple-object-detection/train.js) that [modifies a MobileNet TensorFlow.js Layers model](https://github.com/tensorflow/tfjs-examples/blob/fb430042376431fba804a2495891100f6e2205d6/simple-object-detection/train.js#L66-L101) or [loaded from Google](https://github.com/tensorflow/tfjs-examples/blob/fb430042376431fba804a2495891100f6e2205d6/simple-object-detection/index.js#L146-L147) (TensorFlow.js Layers model as well).

The TFLite model in this folder was created by first converting the TFJS Layer model to a Keras HDF5 model using [the tfjs-converter](https://github.com/tensorflow/tfjs-converter/tree/master/python/tensorflowjs/converters) ([the Python package](https://pypi.org/project/tensorflowjs/1.2.6/)) and then by converting the resulting HDF5 model to a TFLite model using the [TensorFlow Lite converter](https://www.tensorflow.org/lite/convert/).

Because the base mobilenet model [uses functions that aren't in scope by default](https://github.com/keras-team/keras/issues/7431), both the converters have to be patched in order for the conversion to occur successfully. The two patches in this folder take care of this; they're designed to be applied after the `tensorflowjs` and `tensorflow` python packages are installed. To apply them, run `patch -p1 < <path/to/patchfile>` while in the `site-packages` folder of the python installation that was used to install the packages (use a virtualenv!). The patches can be applied in any order.

Note: If your python installation is configured to generate .pyc files on install, be sure to delete the .pyc files for the files that are altered by the patches so that the changed files are actually used (timestamps [_should_ take care of this](https://stackoverflow.com/questions/15839555/when-are-pyc-files-refreshed), but just in case).

### To actually perform the conversion:
  - First download the `model.json` file for the model from the [given URL](https://github.com/tensorflow/tfjs-examples/blob/fb430042376431fba804a2495891100f6e2205d6/simple-object-detection/index.js#L146-L147).
    + Warning: set your `Accept` header or you might get back junk. Or use a browser.
    + `export MODEL_URL="https://storage.googleapis.com/tfjs-examples/simple-object-detection/dist/object_detection_model/model.json"`
    + `curl -H 'Accept: application/json' "${MODEL_URL}" > model.json`
  - Next, grab all the sharded weight files for the model:
    + `export BASE_URL=$(dirname "${MODEL_URL}")`
    + `jq -r '.weightsManifest | .[] | .paths | .[]' model.json | { while read w; do wget "${BASE_URL}/${w}"; done; }`
  - Now, convert to Keras HDF5:
    + `tensorflowjs_converter --input_format tfjs_layers_model --output_format keras model.json simple-object-detection.h5`
  - And finally, to TFLite:
    + `tflite_convert --keras_model_file simple-object-detection.h5 --output_file simple-object-detection.tflite`
