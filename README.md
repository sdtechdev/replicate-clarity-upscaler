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

## Building and Running the Clarity-Upscaler Docker Image Locally
1. Install `replicate-cog` on a local GPU machine:
   ```bash
   sudo curl -o /usr/local/bin/cog -L "https://github.com/replicate/cog/releases/latest/download/cog_$(uname -s)_$(uname -m)"
   sudo chmod +x /usr/local/bin/cog
   ```
   For the latest installation instructions, see [replicate-cog](https://github.com/replicate/cog).

   > **Note:** 
   > While `cog` technically allows deployment from non-GPU machines, deploying this pipeline from a non-GPU machine (e.g., Macbook M3 Pro Max) has been unsuccessful. We recommend using either a personal GPU machine or a [Lambdalabs](https://lambdalabs.com/) GPU instance (A100 40GB) for making changes and pushing the model.

2. Install the `cog` Python package:
   ```bash
   pip install --no-cache-dir cog
   ```
   > **Note:** The cog package is required to build the Docker image using the cog.yaml configuration file

3. Clone this repository:
   ```bash
   git clone https://github.com/sdtechdev/replicate-clarity-upscaler
   cd replicate-clarity-upscaler
   ```

4. Download the required models and checkpoints:
   ```bash
   python download_weights.py
   ```

5. Build the Docker image locally:
   ```bash
   sudo cog build -t clarity-upscale-model
   # Verify the built image using `sudo docker images`
   ```

6. Run the Docker container as a REST API service:
   ```bash
   sudo docker run -d -p 5000:5000 --gpus all clarity-upscale-model
   # Verify the running container using `sudo docker ps`
   ```

7. Test the service by upscaling an image:
   ```python
   import requests
   import base64
   import io
   from PIL import Image

   headers = {'Content-Type': 'application/json'}
   
   data = {
       'input': {
           'image': 'https://i.pinimg.com/736x/9c/50/2c/9c502ca51a038bf0d3c1262ca6f9ae38.jpg'
       }
   }
   
   # Send request and get response
   response = requests.post('http://localhost:5000/predictions', 
                          headers=headers,
                          json=data)
   
   output = response.json()

   # Convert base64 response to image
   base64_image = output['output'][0]
   image = Image.open(io.BytesIO(base64.b64decode(base64_image.split(",")[1])))
   image.show()
   ```