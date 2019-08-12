## TFLite Models

Some TensorFlow Lite Models (`.tflite`, FlatBuffer format) + their sources.

Each top level folder in this repo can contain one or more models and should have a file named `models.toml` with the following structure:
```TOML
group_name = "MobileNet V1"
license = "Apache-2.0" # SPDX Identifier

[[models]] # One of these sections for each model
# These are required:
name = "Mobilenet_V1_1.0_224_quant" # Model Name

# File path relative to the current directory:
# (This can be in a subdirectory, but flat folder structure is encouraged)
file = "mobilenet_v1_1.0_224_quant.tflite"

# These are optional:
type = "quant" # "quant" or "float"
data_type = "u8"
source_url = "https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_2018_08_02/mobilenet_v1_1.0_224_quant.tgz"
version = "0.1.0" # semver compatible version

# Models can also have additional fields:
resolution = 224
scaling = 1.0

# As an example, here's another one:
[[models]]
name = "Mobilenet_V1_1.0_192_quant"
file = "mobilenet_v1_1.0_192_quant.tflite"
type = "quant"
data_type = "uint8"
source_url = "https://storage.googleapis.com/download.tensorflow.org/models/mobilenet_v1_2018_08_02/mobilenet_v1_1.0_192_quant.tgz"
version = "0.1.0"
resolution = 192
scaling = 1.0

# Finally, each `models.toml` file can have a top level readme field with
# information on how the models were created/sourced. This can either
# contain a markdown compatible string (as below) or can contain a file path
# (relative to the directory) prefaced by `file:` where the file being
# pointed to is a markdown file. (i.e. readme = `file:README.md`)
readme = """
These models come straight from [TensorFlow's hosted models](https://www.tensorflow.org/lite/guide/hosted_models).

`mobilenet_v1_1.0_192_quant.tflite` was modified slightly as per patch 0001.
"""
```

Each folder is free to have whatever files are relevant to the model group. The only requirement is that a `models.toml` file be present. Here's an example (in line with the `models.toml` above):
```bash
mobilenet-v1/
├── mobilenet_v1_1.0_192_quant.tflite
├── mobilenet_v1_1.0_224_quant.tflite
├── models.toml
└── patches
    └── 0001-mobilenet_v1_1.0_192_quant-fix.patch
```