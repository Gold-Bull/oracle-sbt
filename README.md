
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

    - Refer [configuration](#configuration) documentation for available options

4. Make sure the user oracle service runs as has read access to the extracted library and configuration


## Configuration

Configuration file uses YAML syntax

### Backup to Azure Block Blob

Refer '[Microsoft Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/scalability-targets#scale-targets-for-blob-storage)' for blob storage limits.

The library stores each oracle backup piece as a single block blob, memory buffer is used to cache the data from oracle database and the data is then written to storage as a single uncommitted block once the buffer is full. 

All the uncommitted blocks will be committed once the data write to backup piece is complete.

The options 'read_buffer_size_mb' & 'write_buffer_size_mb' can be used to adjust the memory buffer, these parameters use 'MiB' as the unit.

The minimum and maximum values allowed for 'read_buffer_size_mb' is 1 and 512.

The minimum and maximum values allowed for 'write_buffer_size_mb' is 1 and 512.

The maximum blob size is equal to 'write_buffer_size_mb * 50K', the Oracle RMAN MAXPIECESIZE should be adjusted according to maximum blob size.

**When the number of blocks written exceeds 50K error will be raised.**

```yaml
---
storage_type: 'block_blob'
block_blob_options:
  storage_url: 'https://<storage account>.blob.core.windows.net/<container name>/<key prefix : optional>/?<sas token>'
  read_buffer_size_mb: 16  # Default read buffer size is 16, minimum value is 1 maximum value is 512.
  write_buffer_size_mb: 8  # Default write buffer size is 8, minimum value is 1 maximum value is 512. Write buffer size dictates the max size of the blob "<write buffer size> * 50000"
```


### Backup to Azure Append Blob

Refer '[Microsoft Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/scalability-targets#scale-targets-for-blob-storage)' for blob storage limits.

The library stores each oracle backup piece as a single append blob, memory buffer is used to cache the data from oracle database and the data is appended to the blob as a single block once the buffer is full.

Append blob will be sealed to make it read-only once the data write to backup piece is complete.

The options 'read_buffer_size_mb' & 'write_buffer_size_mb' can be used to adjust the memory buffer, these parameters use 'MiB' as the unit.

The minimum and maximum values allowed for 'read_buffer_size_mb' is 1 and 512.

The minimum and maximum values allowed for 'write_buffer_size_mb' is 1 and 4. The maximum size of write buffer is limited to 4 MiB because of the blob storage limits.

The maximum blob size is equal to 'write_buffer_size_mb * 50K', the Oracle RMAN MAXPIECESIZE should be adjusted according to maximum blob size.

**When the number of blocks written exceeds 50K error will be raised.**

```yaml
---
storage_type: 'append_blob'
block_blob_options:
  storage_url: 'https://<storage account>.blob.core.windows.net/<container name>/<key prefix : optional>/?<sas token>'
  read_buffer_size_mb: 16  # Default read buffer size is 16, minimum value is 1 maximum value is 512
  write_buffer_size_mb: 4  # Default write buffer size is 4, minimum value is 1 maximum value is 4. Maximum blob size when using append blob is 195 GiB
```


## [RMAN script samples](./RMAN_SAMPLES.md)

TODO


## Known Issues

1. Append blob seal will fail is the storage account has 'hierarchical namespace' enabled.


## [Future Enhancements](https://github.com/Gold-Bull/oracle-sbt/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement+label%3Aenhancement-accepted)
