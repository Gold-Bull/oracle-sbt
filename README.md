## Overview

Oracle SBT for Cloud Storage enables the backup and restoration of Oracle databases to and from cloud storage.

Utilized [Clean-room design](https://en.wikipedia.org/wiki/Clean-room_design) principles to understand the Oracle Media Management Layer (MML) API and developed the corresponding functionality.

This library is useful

- To backup on-premise oracle database to cloud storage
- Cloud native backup solutions does not meet the requirements


## Prerequisites

1. Cloud Storage

    - Azure Blob Storage (both block blob and append blob)

2. Linux

   a. GLIBC 2.17 or later

3. Windows

   a. [Visual C++ Redistributable for Visual Studio 2015-2022](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170#latest-microsoft-visual-c-redistributable-version)


## Installation

**NOTE: Target release date is 14<sup>th</sup> December 2024. Links will be upload post release.**

1. Download the library archive

   a. [Linux]()
   
   b. [Windows]()

2. Extract the library archive to a folder

    - **NOTE**: Do not extract the archive to Oracle installation directories

3. Create configuration file

    - Refer [configuration](./CONFIGURATION.md) documentation for available options

4. Make sure the user oracle service runs as has read access to the extracted library and configuration


## [RMAN script samples](./RMAN_SAMPLES.md)

## [Known Issues](./KNOWN_ISSUES.md)

## [Future Enhancements](https://github.com/Gold-Bull/oracle-sbt/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement+label%3Aenhancement-accepted)
