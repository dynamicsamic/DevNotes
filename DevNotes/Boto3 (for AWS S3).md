#### Client creation
```python
import boto3

s3_client = boto3.client(
	"s3",
    aws_access_key_id=key_id,
	aws_secret_access_key=key_secret,
	endpoint_url=self.endpoint_url, # usually "https://storage.yandexcloud.net/"
	region_name=self.region_name, # usually "ru-central1"
)
```
---
#### Main methods
```python
"""List all objects availavle under certain path"""
bucket_path_metadata = s3_client.list_objects(Bucket='your_bucket', Prefix='path/to/files')

# The response is a dictionary, containing info about objects and metadata about the request. The most important key is "Contents", it contains a list of dictionaries having data related to files. 
# Each file is a dictionary with different attributes, of which the most important are - Key it's the filename with full path (app/import_order_xml/order_1.xml) and Size - it's the file size in bytes. You can use Key to manipulate files on the bucket.  
print(bucket_path_metadata)
{
	"Contents": [
		{
			"Key": "34234",
			"Size": 1233,
			"Etag": "string",
			"LastModified": datetime(...),
			...
		},
		{
			"Key": ...
			...
		}
	
	],
	"Name": "dsfsdf",
	"Prefix": "dfdf",
	...
}
files = bucket_path_metadata.get("Contents")
file_names = [files["Key"] for file in files]

"""Get an object"""
```