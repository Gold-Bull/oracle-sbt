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
