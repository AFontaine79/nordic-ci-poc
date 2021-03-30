# CI/CD proof-of-concept using Nordic SDK
## Overview
This repository represents my solution to Practical Challenge Option 1 of the Dojo Five Technical Interview process.
> Setup a Jenkins, GitHub Actions, or BitBucket Pipelines project using the Nordic SDK.  Describe where to find the .hex file output and the steps that are part of your build pipeline.

This solution uses GitHub Actions.

## Solution
I decided to use the Blinky project from `nRF5_SDK_17.0.2_d674dde`.  I am using a Makefile based build for the nRF52840 from the subdirectory `pca10056/blank/armgcc` subdirectory.  This allows me to use the standard `C/C++ with Make` workflow template for the CI/CD process.

One of the challenges here is having access to the Nordic SDK from the GitHub build servers.  I'm not finding where this is accessible as a repo (or Docker container) within GitHub with associated GitHub Actions.  As such, I have decided to make the SDK part of this repo.

## Copyright
All code files are Copyright (c) 2010 - 2020, Nordic Semiconductor ASA.
