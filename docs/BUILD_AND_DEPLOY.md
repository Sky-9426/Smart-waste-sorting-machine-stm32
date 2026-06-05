# Build & Deploy Guide

This guide outlines the common steps to convert a TFLite model for STM32N6 (INT8), integrate it into the STM32 project, build firmware, and flash the board.

Prerequisites
- `STM32CubeIDE` (matching your board)
- `STM32Cube.AI` or `STEdgeAI` utilities (optional GUI/CLI tools)
- `STM32CubeProgrammer` (for flashing)
- Python 3.8+ with TensorFlow (if you need to re-run TFLite conversion)

Model conversion (high level)
1. If you have a trained Keras/TensorFlow model, produce a TFLite INT8 file using post-training quantization and a representative dataset. Example conversion snippet:

```python
# representative_dataset() yields preprocessed images as uint8/float32 arrays
import tensorflow as tf
converter = tf.lite.TFLiteConverter.from_saved_model('saved_model_dir')
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.representative_dataset = representative_dataset
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.uint8
converter.inference_output_type = tf.uint8
tflite_model = converter.convert()
open('model_int8.tflite','wb').write(tflite_model)
```

2. Verify the `.tflite` runs locally with the TFLite Interpreter on a few samples.

STM32Cube.AI / STEdgeAI conversion
1. Use the STM32Cube.AI plugin in STM32CubeIDE or `stedgeai.exe` to convert `model_int8.tflite` into C sources for the STM32N6 target.
2. In the conversion tool, select target MCU/board (STM32N6570-DK) and enable INT8 kernels where applicable.
3. Export the generated C files into your STM32 project under `application_code/image_classification/STM32N6/` (commonly into a `Model` folder).

Build in STM32CubeIDE
1. Open the project workspace at `application_code/image_classification/STM32N6/` with `STM32CubeIDE`.
2. Ensure include paths reference the generated model headers/sources and any CMSIS-NN kernels.
3. Build the FSBL and Application targets. Fix compile warnings if include paths or macros are missing.

Flash the board
1. Using STM32CubeProgrammer GUI: select the correct ST-LINK, open the generated `.elf` or `.bin` and flash.
2. Command-line example (adjust path to `STM32_Programmer_CLI`):

```powershell
"C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin\STM32_Programmer_CLI.exe" -c port=SWD -d path\to\firmware.bin -v -rst
```

Tips
- Keep model and large artifacts out of Git; add to `.gitignore` (already present).
- If you prefer using the example config, edit `image_classification/config_file_examples/deployment_n6_config_clean.yaml` `model.model_path` to point to your `.tflite` and run the automation tooling (if available).
- For better accuracy on small edge models, use a representative dataset during quantisation and evaluate accuracy drop before deployment.

If you want, I can: (A) add an automated `convert_and_package.py` helper that runs TFLite conversion and packages files for STM32Cube.AI, or (B) create exact STM32CubeProgrammer CLI commands tailored to your firmware files. Which one do you prefer?
