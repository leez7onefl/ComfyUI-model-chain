# ComfyUI Workflows model chaining + highres fix / upscaler + Face fix + Face Swap

![Capture d’écran 2025-03-27 141844](https://github.com/user-attachments/assets/345c1610-b79c-4706-be64-ce78bcae808b)

This project contains a ComfyUI workflow designed to perform :
1. Model chaining between clip text adherence and photorealism
2. Highres Fixes
3. Tiled upscaling
4. Face fixes with loaded model
5. Face Swapping with ReActor
6. Prompts and seeds control

--- 

## Workflow Description

### part 1 - Loaders and samplers :

![Capture d’écran 2025-03-27 141130](https://github.com/user-attachments/assets/d2fae524-b8da-4918-accf-f37bcc353bd9)

#### LoRAs for Base:
- **Load LoRA:** 
  - These nodes load different LoRA models which fine-tune the style of the generated images. 
  
#### Checkpoints and Decoders:
- **Load Checkpoint:** 
  - Used to load specific model checkpoints which contain the weights and biases of the neural network for specific tasks.
- **VAE Decode:**
  - Variational Autoencoder nodes are present for decoding latent vectors back into image space.
  
#### K Samplers:
- **KSampler (Advanced):**
  - These nodes sample latent variables to generate images from the loaded checkpoints, with inputs for positive and negative prompts, noise level, and configuration settings.
  - Parameters like `steps`, `cfg`, and `noise_seed` control the sampling process. The choice of `sampler_name` and `scheduler` affect the trajectory the sampler takes through the latent space.
  
#### Prompts and Configurations:
- **Negative Prompt:**
  - Negative prompts guide the model away from certain features or styles during generation.
- **Refine At Steps / Total Steps:**
  - These integer nodes indicate the total steps for the diffusion process and the point at which refinement should occur, affecting the balance between initial generation and refinement stages.
  
#### CFG (Classifier-Free Guidance):
- **Base CFG / Refine CFG:**
  - Control the strength of guidance during the generation process. Higher values usually result in images more aligned with prompts but may reduce diversity.

#### Latent Interposer & Empty Latent Size Picker:
- **Latent Interposer:**
  - This node is used to switch between different latent representations, feeding them into other parts of the workflow.
- **Empty Latent Size Picker:**
  - Sets the resolution and other parameters for the latent image space where the diffusion model operate

### part 2 - HighRes Fixer :

![image](https://github.com/user-attachments/assets/69f1f07c-9fa7-4276-a38f-4e7848cea797)

#### Additional Negative Prompt:
- **Additional Negative Prompt:**
  - Used for further conditioning the generated output, guiding the model away from undesirable features that might be introduced during upscaling.

#### Upscaling Process:
- **Load Upscale Model:**
  - Loads an upscale model (e.g., `4x_NMKD`) to enhance the resolution of the output image. These models can be specifically trained for upscaling tasks to improve details and clarity.

- **Upscale Image (using Model):**
  - Utilizes the loaded upscale model to refine and enhance the image resolution. The input image is processed through this model to upscale it while trying to maintain or enhance quality.

- **Upscale Image:**
  - An alternative node that upscales images using a specified method like `lanczos`. It’s a standard interpolation technique for resizing images.

#### VAE Processing:
- **VAE Decode:**
  - Decodes images from the latent space after the upscaling has occurred, bringing them back into the image space for final touches and viewing.

- **VAE Encode:**
  - Converts images back into the latent space. This might be used if further processing or noise reduction steps are applied post-upscaling.

#### Additional Parameters:
- **Upscale Image By:**
  - Determines the method of upscaling and the scale factor (`scale_by`), which in this setup is set to `2.0` for doubling the image dimensions.
 
### part 3 - Upscaler : 

![image](https://github.com/user-attachments/assets/ea9dcc7f-23d7-41de-af4a-86f6f6060f24)

#### Upscale Positive Prompt:
- **Upscale Positive Prompt:**
  - Provides additional conditioning when upscaling to enhance desirable features or characteristics during the process.

#### Tiled Upscaling Process:
- **Ultimate SD Upscale:**
  - A specialized node designed for upscaling images piece by piece (tiled), allowing for higher resolution output without resorting to extreme computational resources.
  - **Inputs and Parameters:**
    - **Positive/Negative Engine:** Influences the style and direction of the upscaling process by emphasizing features to include or exclude.
    - **Strength:** Controls the influence of the upscaling model, higher values make the changes more pronounced.
    - **Noise Level/CFG (Classifier-Free Guidance):** Adjusts the balance between quality and adherence to prompts.
    - **Tiling Settings:** 
      - **Tile Width/Height:** Specifies how the image is divided into tiles, width and height parameters ensure efficient and even tile processing.
    - **Padding:** Helps in blending the tiles together smoothly to prevent visible seams.
    - **Overlapping Tiles and Uniformity:** Ensures continuity and smooth transitions between tiles.
    - **Scheduler and Sampler:** Determines the method and trajectory of navigating the latent space during the upscale operation.


### part 4 - Face Fixes : 

![image](https://github.com/user-attachments/assets/77a7d43f-c39a-429b-ba88-52ded9a1db0a)

#### Face Detailing Process:

- **FaceDetailer:**
  - This node refines and enhances facial features within an image.
  - **Inputs and Parameters:**
    - **Image Input:** Takes the input image that needs facial refinement.
    - **Bounding Boxes/Detection:** Identifies and isolates facial regions for targeted enhancement.
    - **Model/CLIP:** Uses a pre-trained model to achieve detailed face enhancements.
    - **Positive/Negative Prompts:** Guides the stylistic enhancement of facial features.
    - **Scheduler/Sampler:** Configured to control the sampling process through latent space using specific methods like `karras`.

#### UltraFastDetectorProvider:
- **Bounding Box/Segmentation Detector:**
  - Utilizes the `boxflow_yolov8s` model to detect face regions, providing precise bounding boxes for targeted detailing.

#### SAMLoader (Impact):
- **Segment Anything Model (SAM):**
  - Loads the SAM model to assist with precise segmentation of face areas.
  - **Device Mode:** Auto mode adjusts the processing depending on available hardware resources.

#### Seed (rghfe):
- **Seed Configuration:**
  - Sets the randomness for the detailing process, allowing for consistent outputs or variety when reprocessing.

#### Additional Parameters:
- **Guide Size/Steps:** Dictates the size and number of steps for guiding the detailing process.
- **Denoise:** Controls the level of noise reduction during detailing to preserve desired textures while maintaining clarity.
- **Expansion/Thresholds:** Adjust settings for bounding box expansion and threshold of significance during face detection.
- **Cycle and Impact Mode:** Iterative process to refine faces through multiple passes if enabled.

### part 5 - Face Swap : 

![Capture d’écran 2025-03-27 141338](https://github.com/user-attachments/assets/0d377c63-85ff-471d-bd51-f57a1225d2da)

#### Load Face Model:
- **ReActor - Load Face Model:**
  - Loads the face model that will be used for the face swap operation. This model is trained to understand facial features and perform swaps.

#### Face Swapping Process:
- **ReActor - Fast Face Swap:**
  - Uses the loaded face model to perform the face-swapping operation on the input image.
  - **Inputs and Parameters:**
    - **Input Image:** The image where the face will be swapped.
    - **Source Image/Face Model:** The image/model from which the face will be taken.
    - **Face Boost:** Enables additional refinement to improve the quality of the swapped face.
    - **Swap Model:** Specifies the swap model being used (`inswapper_128.onnx`), which guides the actual swapping mechanism.
    - **Face Detection:** Uses `retinaface_resnet50` for accurate face detection.
    - **Face Restore Model:** Employs `GFPGANv1.4` for restoring and enhancing facial details post-swap.
    - **Visibility and Weighting:** Adjusts how prominently the swapped face appears with `face_restore_visibility` and `codeformer_weight`.
    - **Gender Detection:** Options to detect gender in the input or source images, though it’s set to "no" in this setup.
    - **Face Indexing:** Determines which faces to swap if there are multiple detected faces in the images.
    - **Console Log Level:** Controls the verbosity of logging for debugging purposes.

### part 6 - Prompts and seeds :

![image](https://github.com/user-attachments/assets/08ab5b92-8504-41d9-b4e7-9afc3b5c30b2)

#### Seed and Prompt Controls:
- **Seed Base & Seed Refine:**
  - These nodes set the random seed values for different stages of the image generation process.
  - The `control_after_generate` set to "fixed" ensures that the output can be reproduced exactly with the same seeds, providing consistency.

- **Positive Prompt Base & Positive Prompt Refine:**
  - These nodes allow for input of textual prompts that positively guide the base generation and refinement stages, respectively.
  - Text added here influences the style and content of the generated images at each stage.

- **Face Prompt:**
  - Provides specific conditioning for facial features, aligning them with desired attributes defined in the prompt text.

#### Preview and Output:

- **Preview Image:**
  - Displays intermediary or final images from the workflow to allow for visual checks of the output. There are nodes for different stages, allowing for iterative feedback.

- **Highres Fix + Tiled Upscale + Face Swap:**
  - This node represents the confluence of the entire workflow where final outputs are generated and maintained.
  - **Filename Prefix:** Allows for naming or organizing the outputs systematically, aiding in identification and storage.

--- 

## Notes
- Ensure enough computational resources are available, especially for video processing.
- Verify file paths and model names are correct to avoid errors during execution.
- Adjust widget values in nodes like face boost settings according to the visual quality desired.

These workflows are designed to streamline the process of face swapping using ComfyUI, enabling efficient and automated model creation and application across different media.

---

**Disclaimer:** 
This workflow is provided "as-is" without any warranties. 
The user assumes all responsibility for compliance with applicable laws and regulations. 
The creators are not liable for any misuse or consequences arising from its use.

---

ReadMe automatically generated with AI
