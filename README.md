# Create a MATLAB Container Image

This repository shows you how to build and customize a Docker&reg; container for MATLAB&reg; and its toolboxes, using the [MATLAB Package Manager (*mpm*)](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/MPM.md). 

You can use this container image as a scalable and reproducible method to deploy and test your MATLAB code.

You can also download pre-built images based on this Dockerfile from [here](https://github.com/mathworks-ref-arch/matlab-dockerfile/pkgs/container/matlab-dockerfile%2Fmatlab).

### Requirements
* [A Running Network License Manager for MATLAB](https://www.mathworks.com/help/install/administer-network-licenses.html)
    * For more information, see [Using the Network License Manager](#use-the-network-license-manager) 
* Linux® Operating System
* Docker
* Git

## Build Instructions

### Get Sources
 
```bash
# Clone this repository to your machine.
git clone https://github.com/mathworks-ref-arch/matlab-dockerfile.git

# Navigate to the downloaded folder.
cd matlab-dockerfile
```

### Build & Run Docker Image
```bash
# Build container with a name and tag of your choice.
docker build -t matlab:r2023a .

# Run the container. Test the container by running an example MATLAB command such as ver.
docker run --init --rm -e MLM_LICENSE_FILE=27000@MyServerName matlab:r2023a -batch ver
```
The [Dockerfile](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/Dockerfile) defaults to building a container for MATLAB R2023a.

The example command `ver` displays the version number of MATLAB and other installed products. For more information, see [ver](https://www.mathworks.com/help/matlab/ref/ver.html). For more information on running the container, see the section on [Running the Container](#run-the-container).

> **Note**
>
> Using the `--init` flag in the `docker run` command ensures that the container stops gracefully when a `docker stop` or `docker kill` command is issued.
> For more information, see the following links:
> * [Docker run reference page](https://docs.docker.com/engine/reference/run/#specify-an-init-process).
> * [Blog post on the usage of init](https://www.baeldung.com/ops/docker-init-parameter).


## Customize the Image

By default, the [Dockerfile](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/Dockerfile) installs MATLAB for the latest available MATLAB release without any additional toolboxes or products in the `/opt/matlab/${MATLAB_RELEASE}` folder.

Use the options below to customize your build.

### Customize MATLAB Release, MATLAB Product List, MATLAB Install Location and License Server
The [Dockerfile](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/Dockerfile) supports the following Docker build-time variables:

| Argument Name | Default value | Description |
|---|---|---|
| [MATLAB_RELEASE](#build-an-image-for-a-different-release-of-matlab) | r2023a | The MATLAB release you want to install, in lower-case. For example: `r2019b`|
| [MATLAB_PRODUCT_LIST](#build-an-image-with-a-specific-set-of-products) | MATLAB | Products to install as a space-separated list. For more information, see [MPM.md](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/MPM.md). For example: `MATLAB Simulink Deep_Learning_Toolbox Fixed-Point_Designer`|
| [MATLAB_INSTALL_LOCATION](#build-an-image-with-matlab-installed-to-a-specific-location) | /opt/matlab/r2023a | The path to install MATLAB. |
| [LICENSE_SERVER](#build-an-image-with-license-server-information) | *unset* | The port and hostname of the machine that is running the Network License Manager, using the `port@hostname` syntax. For example: `27000@MyServerName` |

Use these arguments with the the `docker build` command to customize your image.
Alternatively, you can change the default values for these arguments directly in the [Dockerfile](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/Dockerfile).

#### Build an Image for a Different Release of MATLAB
For example, to build an image for MATLAB R2019b, use this command.
```bash
docker build --build-arg MATLAB_RELEASE=r2019b -t matlab:r2019b .
```

#### Build an Image with a specific set of products
For example, to build an image with MATLAB and Simulink, use this command.
```bash
docker build --build-arg MATLAB_PRODUCT_LIST='MATLAB Simulink' -t matlab:r2023a .
```

#### Build an Image with MATLAB installed to a specific location
For example, to build an image with MATLAB installed at /opt/matlab, use this command.
```bash
docker build --build-arg MATLAB_INSTALL_LOCATION='/opt/matlab' -t matlab:r2023a .
```

#### Build an Image with License Server Information

Including the license server information with the `docker build` command means you do not have to pass it when running the container.
```bash
# Build container with the License Server.
docker build --build-arg LICENSE_SERVER=27000@MyServerName -t matlab:r2023a .

# Run the container, without needing to pass license information.
docker run --init --rm matlab:r2023a -batch ver
```

## Use the Network License Manager
This container requires a Network License Manager to license and run MATLAB. You will need either the port and hostname of the Network License Manager, or a `network.lic` file.

**Step 1**: Contact your system administrator, who can provide one of the following:

1. The address to your server, and the port it is running on. 
    For example: `27000@MyServerName.example.com`

2. A `network.lic` file which contains the following lines:
    ```bash
    # Sample network.lic
    SERVER MyServerName.example.com <optional-mac-address> 27000
    USE_SERVER
    ```

3. A `license.dat` file. Open the `license.dat` file, find the `SERVER` line, and either extract the `port@hostname`, or create a `network.lic` file by copying the `SERVER` line and adding a `USE_SERVER` line below it.

    ```bash
    # snippet from sample license.dat
    SERVER MyServerName.example.com <mac-address> 27000
    ```
---
**Step 2**: Use `port@hostname` or the `network.lic` file with either the `docker build` **or** the `docker run` command.

With the `docker build` command, either:

- Specify the `LICENSE_SERVER` build-arg.

    ```bash
    # Example
    docker build -t matlab:r2023a --build-arg LICENSE_SERVER=27000@MyServerName .
    ```
- Use the `network.lic` file:
    1. Place the `network.lic` file in the same folder as the Dockerfile.
    1. Uncomment the line `COPY network.lic /opt/matlab/licenses/` in the Dockerfile.
    1. Run the docker build command **without** the `LICENSE_SERVER` build-arg:

    ```bash
    # Example
    docker build -t matlab:r2023a .
    ```
    
With the `docker run` command, use the `MLM_LICENSE_FILE` environment variable. For example:

```bash
docker run --init --rm -e MLM_LICENSE_FILE=27000@MyServerName matlab:r2023a -batch ver
```

## Run the Container
If you did not provide the license server information when building the image, then provide it when running the container. Set the environment variable `MLM_LICENSE_FILE` using the `-e` flag, with the  network license manager's location in the format `port@hostname`.

```bash
# Start MATLAB, print version information, and exit:
docker run --init --rm -e MLM_LICENSE_FILE=27000@MyServerName matlab:r2023a -batch ver
```

You can run the container **without** specifying `MLM_LICENSE_FILE` if you provided the license server information when building the image, as shown in the examples below.

### Run MATLAB in an Interactive Command Prompt
To start the container and run MATLAB in an interactive command prompt, execute:
```bash
docker run --init -it --rm matlab:r2023a
```
### Run MATLAB in Batch Mode
To start the container, run a MATLAB command, and then exit, execute:
```bash
# Container runs the command RAND in MATLAB and exits.
docker run --init --rm matlab:r2023a -batch rand
```

### Run MATLAB with Startup Options
To override the default behavior of the container and run MATLAB with any set of arguments, such as `-logfile`, execute:
```bash
docker run --init -it --rm matlab:r2023a -logfile "logfilename.log"
```
To learn more, see the documentation: [Commonly Used Startup Options](https://www.mathworks.com/help/matlab/matlab_env/commonly-used-startup-options.html).


## More MATLAB Docker Resources
* Explore pre-built MATLAB Docker Containers on Docker Hub: https://hub.docker.com/r/mathworks
    * [MATLAB Containers on Docker Hub](https://hub.docker.com/r/mathworks/matlab) hosts container images for multiple releases of MATLAB.
    * [MATLAB Deep Learning Containers on Docker Hub](https://hub.docker.com/r/mathworks/matlab-deep-learning) hosts container images with toolboxes suitable for Deep Learning.

* This Dockerfile builds on the matlab-deps container image and installs MATLAB. For other possibilities,
see the examples in the [**alternates folder**](alternates) of this repository:
    * [matlab-installer](alternates/matlab-installer) is an example of a Dockerfile that uses the matlab installer rather than mpm to install MATLAB in the container, allowing the installation of some toolboxes that are not currently supported by mpm.

* Enable additional capabilities using the [MATLAB Dependencies repository](https://github.com/mathworks-ref-arch/container-images/tree/master/matlab-deps). 
For some workflows and toolboxes, you must specify dependencies. You must do this if you want to do any of the following tasks:
    * Install extended localization support for MATLAB.
    * Play media files from MATLAB.
    * Generate code from Simulink.
    * Use mex functions with gcc, g++, or gfortran.
    * Use the MATLAB Engine API for C and Fortran.
    * Use the Polyspace 32-bit tcc compiler.
    
    The [matlab-deps repository](https://github.com/mathworks-ref-arch/container-images/tree/master/matlab-deps) repository lists Dockerfiles for various releases and platforms. [Click here to view the Dockerfile for R2023a](https://github.com/mathworks-ref-arch/container-images/blob/master/matlab-deps/r2023a/ubuntu20.04/Dockerfile).

    These Dockerfiles contain commented lines with the libraries that support these additional capabilities. Copy and uncomment these lines into your Dockerfile.

## Help Make MATLAB Even Better
You can help improve MATLAB by providing user experience information on how you use MathWorks products. Your participation ensures that you are represented and helps us design better products. To opt out of this service, delete the following line in the Dockerfile:
```Dockerfile
ENV MW_DDUX_FORCE_ENABLE=true MW_CONTEXT_TAGS=MATLAB:DOCKERFILE:V1
```

To learn more, see the documentation: [Help Make MATLAB Even Better - Frequently Asked Questions](https://www.mathworks.com/support/faq/user_experience_information_faq.html).

## Feedback
We encourage you to try this repository with your environment and provide feedback. If you encounter a technical issue or have an enhancement request, create an issue [here](https://github.com/mathworks-ref-arch/matlab-dockerfile/issues).

----

Copyright (c) 2021-2023 The MathWorks, Inc. All rights reserved.

----
