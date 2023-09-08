# IRIS-HEP-S3 Project Notes

## Weeks 1-3

#### Rucio S3 Interface should have the following features:</br>
Interacting with the bucket should be possible after updating them using external clients
```bash
mc ls user.lheinric:someuniquename 
mc cp file.root user.lheinric:someuniquename/file.root
```
> Should be not problematic, except for authentication of `mc`. </br>
 S3-endpoint, access key, secret key should also be extracted. API signature defaults to S3V4.

```bash
rucio s3 make-bucket user.lheinric:someuniquename
```
>`make-bucket` as well as other functions can be found in minio’s python package, or directly called using the boto3
Should the syntax be translated, or S3 that CERN uses somehow allows the scope syntax from Rucio?
`user.lheinric` would be the name of the bucket, and there can be many other buckets like `user.kmeliush`, `user.dquijote`. Then each bucket would have a subfolder like `user.kmeliush/data`


```bash
rucio download user.lheinric:someuniquename/file.root
```
>Download command already exists as part of rucio, but there most probably will be some compatibility issues.

Since rucio is already authenticated, connection and credentials extraction should be possible:
```
rucio s3 credentials > ~/.mc/config.json
```


Things to keep in mind:
* check python minio package for mc’s functionality
* different backend compatibility 
* lightweight global namespace so that users don’t have to bother about specific storage backends?
* bucket policies in S3



## Weeks 4-5

#### Creating client (class) for the S3 interface:

Should be derived from [baseclient.py](https://github.com/rucio/rucio/blob/master/lib/rucio/client/baseclient.py) and can be based on some other connection clients. </br>

draft:
```python
    class S3Client(BaseClient):
        def __init__(self, resource='s3', endpoint_url='https://localhost:9000', key_id='admin', secret_key='password', config=Config(signature_version='s3v4'), region="?"):
            super().__init__()
            self.s3client = boto3.client(resource, endpoint_url=endpoint_url, aws_access_key_id=key_id, aws_secret_access_key=secret_key, config=config, region_name=region)

```



> Extracting authentication: [rucio/client/baseclient.py](https://github.com/rucio/rucio/blob/7ccac3dcf98af3ccd383478c52c3e7fca2752527/lib/rucio/client/baseclient.py#L73)

```python
def get_signed_url(rse_id: str, service: str, operation: str, url: str, lifetime=600) -> str:
```
```python
class CredentialClient(BaseClient):
	"""Credential client class for working with URL signing"""

	CREDENTIAL_BASEURL = 'credentials'
```
>Also linked to credentials/url signing

ADDITIONAL:
- 
  	- MINIO_ROOT_USER=admin
  	- MINIO_ROOT_PASSWORD=password
>minio_access_key, minio_secret_key is deprecated, update in docker-compose.yml files


## Weeks 6-9


### Major: 
Redo the project architecture, get rid of `rucio s3` interface, use direct commands:

```sh
rucio s3 make-bucket user.kmeliush:someuniquename -> rucio bucket create user.mlassnig:/analysis1/temp/
```

With additional functionality
```sh
rucio bucket upload --from localdirectory/ --to user.mlassnig:/analysis1/temp/
```

### Minor:

```lib/rucio/common/exception.py::216``` - Unable to to load user credentials 



## Weeks 9-12:
List of arguments to set group permissions when uploading files:
https://boto3.amazonaws.com/v1/documentation/api/latest/reference/customizations/s3.html#boto3.s3.transfer.S3Transfer.ALLOWED_UPLOAD_ARGS


```
s3.upload_file(
    'FILE_NAME', 'BUCKET_NAME', 'OBJECT_NAME',
    ExtraArgs={'ACL': 'public-read'}
)
```

### Managing cache:
```

def write_to_cache(key, value):
    
    Set a value on a key in a cache.

    :param key: Key that stores the value.
    :param value: Value to be stored.
    """
    key = key.replace(' ', '')
    REGION.set(key, value)


def delete_from_cache(key):
    """
    Delete from cache any data stored for the given key

    :param key: Key that stores the value.
    """
    key = key.replace(' ', '')
    REGION.delete(key)

```