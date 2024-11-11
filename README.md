# Clarity-Upscaler

This repository contains the code for the Clarity-Upscaler deployment. The initial codebase was taken and modified from https://github.com/philz1337x/clarity-upscaler/tree/main.
- See [PR: Ar-289-deploy](https://github.com/sdtechdev/replicate-clarity-upscaler/pull/1)

## Deploying the Clarity-Upscaler Model on Replicate

1. Install `replicate-cog` on a local GPU machine:
   ```bash
   sudo curl -o /usr/local/bin/cog -L "https://github.com/replicate/cog/releases/latest/download/cog_$(uname -s)_$(uname -m)"
   sudo chmod +x /usr/local/bin/cog
   ```
   For the latest installation instructions, see [replicate-cog](https://github.com/replicate/cog).

   > **Note:** 
   > While `cog` technically allows deployment from non-GPU machines, deploying this pipeline from a non-GPU machine (e.g., Macbook M3 Pro Max) has been unsuccessful. We recommend using either a personal GPU machine or a [Lambdalabs](https://lambdalabs.com/) GPU instance (A100 40GB) for making changes and pushing the model.

2. Clone this repository:
   ```bash
   git clone https://github.com/sdtechdev/replicate-clarity-upscaler
   cd replicate-clarity-upscaler
   ```

3. Download the required models and checkpoints:
   ```bash
   python download_weights.py
   ```

   To test if the pipeline is working properly, make a prediction locally:
   ```bash
   sudo cog predict -i image=@init.png
   ```

4. Either create a new model on the [sdtechdev](https://replicate.com/sdtechdev) Replicate account or use the existing model repository: [clarity-upscaler-4-deployment](https://replicate.com/sdtechdev/clarity-upscaler-4-deployment/).

5. Push the model to Replicate:
   ```bash
   # First, login to Replicate
   sudo cog login

   # Then push the model
   sudo cog push r8.im/sdtechdev/clarity-upscaler-4-deployment
   ```
   Building and pushing the Docker image (approximately 27GB) can take 10-12 minutes depending on available cache images and internet speed.

**Note:** A GitHub Actions workflow is available to automate model deployment to Replicate (see `.github/workflows/push.yml`). However, due to the large Docker image size, the workflow fails on default GitHub runners. A custom-hosted runner is required to successfully push using GitHub Actions.