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

1. Download the library archive

   a. [Linux](https://sasgrsv2cind59037.blob.core.windows.net/oracle-sbt/1.0.0.0/gbosfcs-1.0.0-linux-x64.tar.gz)
   
   b. [Windows](https://sasgrsv2cind59037.blob.core.windows.net/oracle-sbt/1.0.0.0/gbosfcs-1.0.0.0-windows-x64.zip)

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

**When the number of blocks written exceeds 50K error will be raised.**

Blob maximum size will be equal to 50K times the 'BLKSIZE' parameter in Oracle RMAN. For default 'BLKSIZE' value 256KiB the maximum blob size is 12GiB, after reaching the data upload reaches the maximum blob size error will be returned.

To increase the maximum blob size consider increasing the 'BLKSIZE', recommended 'BLKSIZE' is 8MiB per channel. Refer "[Configure block size in RMAN](#configure-block-size-in-rman)" to adjust the 'BLKSIZE'

MAX BLOB SIZE = 50000 * BLKSIZE

```yaml
---
storage_type: 'block_blob'
block_blob_options:
  storage_url: 'https://<storage account>.blob.core.windows.net/<container name>/<key prefix : optional>/?<sas token>'
```


### Backup to Azure Append Blob

Refer '[Microsoft Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/scalability-targets#scale-targets-for-blob-storage)' for blob storage limits.

The library stores each oracle backup piece as a single append blob, memory buffer is used to cache the data from oracle database and the data is appended to the blob as a single block once the buffer is full.

Append blob will be sealed to make it read-only once the data write to backup piece is complete.

**When the number of blocks written exceeds 50K error will be raised.**

Blob maximum size will be equal to 50K times the 'BLKSIZE' parameter in Oracle RMAN. For default 'BLKSIZE' value 256KiB the maximum blob size is 12GiB, after reaching the data upload reaches the maximum blob size error will be returned.

To increase the maximum blob size consider increasing the 'BLKSIZE', recommended 'BLKSIZE' is 4MiB per channel.

For backup operation 4MiB is the maximum allowed 'BLKSIZE' when using append blob because of the storage limits.

MAX BLOB SIZE = 50000 * BLKSIZE

```yaml
---
storage_type: 'append_blob'
block_blob_options:
  storage_url: 'https://<storage account>.blob.core.windows.net/<container name>/<key prefix : optional>/?<sas token>'
```


## [RMAN samples](./RMAN_SAMPLES.md)

Make sure "&lt;library location full path&gt;" is replaced with the full path to the library location including the library file name and "&lt;parameter file path&gt;" is replaced with full path to the parameters file.

### Configuring SBT parameters on each channel

```
connect target /;
RUN
{
ALLOCATE CHANNEL c1 DEVICE TYPE SBT_TAPE PARMS='SBT_LIBRARY=<library location full path>,ENV=(GB_SBT_PARAM_FILE=<parameter file path>)';
ALLOCATE CHANNEL c2 DEVICE TYPE SBT_TAPE PARMS='SBT_LIBRARY=<library location full path>,ENV=(GB_SBT_PARAM_FILE=<parameter file path>)';

BACKUP AS BACKUPSET DATABASE FORMAT '/%d/%Y/%M/%D/%U';

RELEASE CHANNEL c1;
RELEASE CHANNEL c2;
}
```

### Configuring Automatic SBT Channels

  - Login to RMAN

  - Connect to target

  ```
  connect target /;
  ```

  - Configure SBT parameters

  ```
  CONFIGURE CHANNEL DEVICE TYPE SBT_TAPE PARMS 'SBT_LIBRARY=<library location full path>,ENV=(GB_SBT_PARAM_FILE=<parameter file path>)';
  ```

  - Set SBT as the default device type

  ```
  CONFIGURE DEVICE TYPE SBT_TAPE PARALLELISM 2;
  ```

  - Run backup operation

  ```
  BACKUP AS BACKUPSET DATABASE FORMAT '/%d/%Y/%M/%D/%U';
  ```

### Configure block size in RMAN

  - On each allocated channel

  ```
  connect target /;
  RUN
  {
  ALLOCATE CHANNEL c1 DEVICE TYPE SBT_TAPE PARMS='SBT_LIBRARY=<library location full path>,ENV=(GB_SBT_PARAM_FILE=<parameter file path>),BLKSIZE=8388608';
  ALLOCATE CHANNEL c2 DEVICE TYPE SBT_TAPE PARMS='SBT_LIBRARY=<library location full path>,ENV=(GB_SBT_PARAM_FILE=<parameter file path>),BLKSIZE=8388608';

  BACKUP AS BACKUPSET DATABASE FORMAT '/%d/%Y/%M/%D/%U';

  RELEASE CHANNEL c1;
  RELEASE CHANNEL c2;
  }
  ```

  - For Automatic SBT channels

  ```
  CONFIGURE CHANNEL DEVICE TYPE SBT_TAPE PARMS 'SBT_LIBRARY=<library location full path>,ENV=(GB_SBT_PARAM_FILE=<parameter file path>),BLKSIZE=8388608';
  ```


## Known Issues

1. Oracle RMAN Proxy Copies are not supported.
2. Append blob seal will fail is the storage account has 'hierarchical namespace' enabled.


## [Future Enhancements](https://github.com/Gold-Bull/oracle-sbt/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement+label%3Aenhancement-accepted)
