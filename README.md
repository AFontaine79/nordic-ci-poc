# CI/CD proof-of-concept using Nordic SDK
## Overview
This repository represents my solution to Practical Challenge Option 1 of the Dojo Five Technical Interview process.
> Setup a Jenkins, GitHub Actions, or BitBucket Pipelines project using the Nordic SDK.  Describe where to find the .hex file output and the steps that are part of your build pipeline.

This solution uses GitHub Actions.


## Copyright
All code files are Copyright (c) 2010 - 2020, Nordic Semiconductor ASA.


## Solution
### Build
I decided to use the Blinky project from `nRF5_SDK_17.0.2_d674dde`.  I am using a Makefile based build for the nRF52840 from the subdirectory `pca10056/blank/armgcc` subdirectory.  This allows me to use the standard `C/C++ with Make` workflow template for the CI/CD process.

One of the challenges here is having access to the Nordic SDK from the GitHub build servers.  I'm not finding where this is accessible as a repo (or Docker container) within GitHub with associated GitHub Actions.  As such, I have decided to make the SDK part of this repo.

There were two issues I needed to resolve to get the bulid working.  The first was that the make process was unable to find the GNU ARM compiler installed by the previous step of the CI/CD flow.  The second was that the posix makefile uses a different GCC version than the Windows makefile.  This took some experimentation to understand the issues and get them resolved properly.

For the first, it seems that [fiam/arm-none-eabi-gcc@v1](https://github.com/fiam/arm-none-eabi-gcc) is putting the installation in the `/tmp` folder, not the location assumed by the posix makefile.  The simple solution here was to comment out the assignment to `GNU_INSTALL_ROOT` in the posix makefile causing GNU make to treat as an empty string, which in turn causes the shell to search `PATH` for the location of `arm-none-eabi-gcc`.  This works.

For the second issue, it was necessary to change the release specified to `fiam/arm-none-eabi-gcc@v1` so that it matches the one in the posix makefile.


### Deployment
I initially explored Docker for the deployment.  However, after doing some reading and going through the Docker setup and tutorial, I started to get the feeling that this was the wrong approach.  A Dockerized app is a self-sufficient container that can run on the cloud or on-premises.  This is similar to a virtual machine and in fact the Microsoft documentation [makes exactly this comparison](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/container-docker-introduction/docker-defined#comparing-docker-containers-with-virtual-machines).  My understanding for this use case is that we only need to deploy a single hex file that can be loaded onto a board.

There is a question here of whether continuous deployment is only for select testers and pre-release (i.e. as for an agile client delivery schedule during development) or for deployment to the field (i.e. builds should post directly to a firmware update server).  I am assuming only the former as it is my understanding that only tagged releases should be used for production (although I am open to other philosophies).  For this reason, I have decided to host ony the hex file and operate with the assumption that the recipient (i.e. client performing validation tests) has been provided a method for updating their devices.

I searched for some tutorials or guidelines for hosting a single file to a Docker Hub and found [this post](https://tonyuk.medium.com/how-to-build-a-ci-cd-pipeline-with-go-github-actions-and-docker-3c69e50b6043) using the Go language for their example.  Unfortunately, this example is still performing the build inside Docker rather than within the GitHub workflow.  There is a seeming incongruency here as building inside Docker would make GitHub's default `C/C++ with Make` workflow pointless.  Why do the build on GitHub if it also needs to be done inside Docker?  It is difficult for me to understand the degree to which there are standard practices for setting up these flows vs it being up to individual discretion.

I decided to abandon the Docker route and see if there were any GitHub Actions for posting files to standard file hosting services.  I did not find them for OneDrive or iCloud and I checked if OneDrive supports FTP (it does not appear to).  Dropbox and Google Drive, however, have GitHub upload actions.  I explored the APIs on both of these and how to get an auth token, ultimately deciding on Dropbox for this project.  For this, I created a Dropbox "app" whose sole purpose is to allow authenticated upload of files into a shared folder.

The upload step of the GitHub workflow uses [deka0106/upload-to-dropbox@v2](https://github.com/deka0106/upload-to-dropbox).  It is uploading to a shared subfolder of the Dropbox app folder.  The access link provides access to the folder, not a specific file.  The file contents can change and be updated without breaking the link.

Get the deployment file [here](https://www.dropbox.com/sh/7kqgtj6nyr00g28/AAAqOnMRULmM28KYPjGqx_Cka?dl=0).


## Stages of build pipeline
The CI/CD flow is triggered on any push or pull request occurring on `main` and is specified in [c-cpp.yml](https://github.com/AFontaine79/nordic-ci-poc/blob/main/.github/workflows/c-cpp.yml).  There are four stages.

1. Perform checkout - `uses: actions/checkout@v2` - Standard procedure for all repos.
2. Install ARM GCC - `uses: fiam/arm-none-eabi-gcc@v1` - Required for a cross-compilation build of the Nordic project.
3. Perform build - `run: make` - Runs the build and places the output .bin and .hex files in /_build.
4. Deploy to Dropbox - `uses: deka0106/upload-to-dropbox@v2` - Uploads the hex file to shared Dropbox folder.


## Suggestions for future work
- Add versioning and tie it into the build process so that, at the least, each file is uploaded with a different build number.  The version number (including build number) should both be part of the filename and stored inside the application.
- Create a separate deployment job, move the Upload to Dropbox action there, and make that job dependent on success of the build job.
- Add unit tests
- Use a build matrix to verify the build on Ubuntu, Windows, and Mac OS, using only one of those results for deployment.  (It is unclear whether this adds any value since this a cross-compilation, not a build designed to run on any of those OSes.  As such, the value of this suggestion seems dubious and I recommend against it.)
