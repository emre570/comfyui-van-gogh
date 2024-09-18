
# ComfyUI Van Gogh Pipeline

### AppNation AI Engineer Case Study Report

  

## Introduction

This report describes a custom image generation pipeline built using ComfyUI. Also includes installation steps, pipeline details and some common troubleshooting. The pipeline takes an input image and combines the image’s style based on Vincent van Gogh’s image’s style, while maintaining the composition from the input image.

## Installation

This section contains installation of ComfyUI, ComfyUI Manager for installing other related packages.

### ComfyUI

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdO6M-JdxbvNUU8z53_NC9Hh_gSNDJY8O7u3NVMgFuo-wk94U2IajjVg-nLU16i24RRupQ-fFZZ1erihkSCeWqAMtN6bYtl0RfJ_Yf0aWNsb5I-GRh5VtUChjHPJ8HL6AVcKji8E06Q1KE8-4e4IEEPwOEC?key=e8Yl9OKovREwSnhn2_3RqQ)

1. Download the latest release of ComfyUI from its [GitHub page](https://github.com/comfyanonymous/ComfyUI/releases).  

2. Extract the zip file to a folder.

3. If you have an NVIDIA CUDA GPU, start run_nvidia_gpu.bat file. If not, start run_cpu.bat for CPU.

Now you have ComfyUI ready to go. But, we are not done yet.
    

### ComfyUI Manager

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXengLD5PIsRzUSD54mPCfLqsJCrT4Tsoc4_SxsBqeuxjmbgy_sNcS3k9ArQi8JZT1_moDOxw2wcvRYnCiEWUGagBPg0XUUqW1atj6k0j6O1B1EulAc0D1RUlE9-L5PrcW1Kz5r8GzqgmILE_eRUvUVRukL2?key=e8Yl9OKovREwSnhn2_3RqQ)

ComfyUI Manager allows us to easily manage our packages, models and many more. To install, we must follow these steps:

1.  Head to ComfyUI Manager’s GitHub page.
    
2.  Go to the Installation section.
    
3.  Follow the steps as written.
    
4.  Restart ComfyUI if running.
    
5.  Now you have ComfyUI Manager installed and ready to use.
    

### Pipeline

1.  Download and load the van-gogh.json file from ComfyUI.
    
2.  If you see red nodes, open the ComfyUI Manager Menu, then head to the “Install Missing Custom Nodes” section.
    
3.  Install the missing packages, then restart ComfyUI.
    

### Models-Packages

Before we use the pipeline, we must install related models and packages which are:

  

**Packages:**

-   [ComfyUI_IPAdapter_plus](https://github.com/cubiq/ComfyUI_IPAdapter_plus)  for IPAdapter,
    

-   Open ComfyUI Manager Menu
    
-   Open “Custom Nodes Manager” Menu
    
-   From this menu, you can download any node packages you want from this menu.
    
-   Download the mentioned package and restart ComfyUI.
    

-   [ComfyUI's ControlNet Auxiliary Preprocessors](https://github.com/Fannovel16/comfyui_controlnet_aux)  for ControlNet,
    

-   Open ComfyUI Manager Menu
    
-   Open “Custom Nodes Manager” Menu
    
-   Download the mentioned package and restart ComfyUI.
    

**Models:**

-   [sd_xl_base_1.0.safetensors](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0)  for SDXL model,
    

	-   Open ComfyUI Manager Menu
	    
	-   Open “Model Manager” Menu
	    
	-   From this menu, you can download any model you want from this menu.
	    
	-   Set type to checkpoint, and set base to SDXL.
	    
	-   Download the mentioned model and restart ComfyUI.
    

-   [stabilityai/control-lora-canny-rank128.safetensors](https://huggingface.co/stabilityai/control-lora)  for ControlNet,
    

	-   Open ComfyUI Manager Menu
	    
	-   Open “Model Manager” Menu
	    
	-   Set type to controlnet, and set base to SDXL.
	    
	-   Download the mentioned model and restart ComfyUI.
    

-   [ip-adapter-plus_sdxl_vit-h.safetensors](https://huggingface.co/h94/IP-Adapter)  for IPAdapter model
    

	-   Open ComfyUI Manager Menu
	    
	-   Open “Model Manager” Menu
	    
	-   Set type to IP-Adapter, and set base to SDXL.
	    
	-   Download the mentioned model and restart ComfyUI.
    

Now you should be ready.

## Pipeline Details

The pipeline involves multiple processes such as loading and resizing images, applying conditioning with ControlNet, generating latent representations using a VAE, and editing the output using models like IPAdapter. The report breaks down the individual nodes, their connections, and the transformations applied throughout the pipeline. It also includes performance notes and suggestions for optimal operation.

We will now explain key nodes of the pipeline. Let’s start:

----------

### 1. ControlNet

The pipeline uses the ControlNet model, which is loaded through this node. The ControlNetLoader node prepares the Canny model to guide the image generation process by detecting edges.

**Observations:**

-   If you use a prompt like "Style of Vincent Van Gogh," the model will attempt to generate an image that mimics Van Gogh's artistic style. In this case, without specific positive or negative prompts, the model's interpretation might be overly influenced by this style. It’s important to carefully adjust the prompts for precise control over the output.
    

----------

### 2. Image Preprocessing and Resizing

-   **LoadImage:** Two images are loaded into the pipeline. One image for input, another from using its style.
    
**ImageResize+:**
    

-   Width is set to 960 pixels, and height is set to 0 with the option to keep proportion. This means the width is fixed at 960 pixels while the height adjusts dynamically to maintain the image's aspect ratio. This can be adjusted freely.
    
-   Interpolation method is nearest, which performs basic resizing but may not produce smooth results when upscaling or downscaling large amounts.
    

-   The resizing step standardizes the image dimensions before feeding it into subsequent nodes like VAE encoding and Canny edge detection.
    

----------

### 3. Edge Detection with Canny

Canny  processes the resized image to detect edges, creating an edge map that ControlNet uses for guidance.

Thresholds are set to 0.03 (low) and 0.25 (high), which balances between detecting weaker edges and ignoring very subtle image details. These settings help to capture clear, distinct outlines from the image while ignoring noise.

**Observations:**

Canny's parameters should be carefully adjusted to ensure accurate edge detection that reflects the image’s details without over- or under-exaggerating the outlines. Based on experimentation, the ranges of 0.03-0.05 for the low threshold and 0.25-0.35 for the high threshold yielded better edge results.

----------

### 4. IPAdapter for Style and Composition Editing

-   **IPAdapterUnifiedLoader:**
    

	-   Loads the IPAdapter model (PLUS (high strength)) to perform image style and composition-based edits. This is connected to the IPAdapterStyleComposition node.
    

-   **IPAdapterStyleComposition:**
    

	-   Combines different elements like the original image, style guidance, and IPAdapter model to modify the style and composition of the output image.
	    
	-   The inputs include:
	    

	-   Model and IPAdapter
	    
	-   Image Style from the resized input image.
	    
	-   Image Composition based on the outputs of the VAE and KSampler.
	    
	-   Image Negative allows you to further fine-tune the composition by specifying what to avoid.
    

----------

### 5. Sampling with KSampler

-   **KSampler:**
    

	-   The KSampler is a crucial part of the diffusion process, where it refines the latent space representations and generates the final image. It uses the dpmpp_2m sampling method and Karras noise scheduling to balance the noise reduction and sharpness during the sampling process.
    

**Observations:**

With a NVIDIA GeForce RTX 3070 Ti (8GB VRAM, 6144 CUDA cores), running 20 sampling steps with the KSampler takes approximately 25 seconds. This performance note helps set expectations for processing time during image generation, ensuring the workflow remains efficient.

----------

## Potential Improvements, Future Enhancements And Problems

-   **Experiment with different ControlNet models:** You could try different ControlNet models like depth or pose models to see how they affect the structural guidance in your image generation.
    
-   **Tuning sampling parameters:** Changing the KSampler's settings, such as increasing the number of steps or adjusting the CFG scale, could yield different levels of image sharpness and fidelity.
    
-   **Further edge detection exploration:** Experimenting with different Canny threshold values might enhance or reduce the influence of edge guidance, depending on your goals for structural accuracy.
- **Poor Edge Detection**: The edges detected by the Canny edge detection node are too weak or too strong, leading to poor results.
	-   **Solution**:
	    -   **Fine-tune the thresholds**: Adjust the **low and high thresholds** in the Canny node to values that better capture the important details of the image. Common ranges are:
	        -   Low threshold: `0.03` to `0.05`
	        -   High threshold: `0.25` to `0.35`
	    -   **Experiment with lighting**: If the image has extreme lighting or contrast, the edge detection may behave unexpectedly. Adjust the brightness/contrast of the input image before passing it to the Canny node.

- **Incompatible Model Types**: The output image appears incorrect or the pipeline fails when combining different models like IPAdapter, ControlNet, and diffusion models.
	-   **Solution**:
	    -   **Use models of the same type**: For instance, **IPAdapter Style & Composition SDXL** works only with **SDXL models**, and the same applies to **ControlNet** models. This is because different model types (e.g., SD1.5, SD2.1, SDXL) have different architectures and latent spaces.
	    -   **Why**: Each model type (e.g., SDXL or SD1.5) processes images and latent representations differently. If you mix incompatible models, their latent spaces will not align, causing mismatches during generation. For example, an SD1.5 ControlNet model won’t work properly with an SDXL diffusion model, as they expect different input formats and operate on different scales.
