# curl Examples

Here are some examples of sending API requests to Curiefense using curl. 

This list is not meant to be exhaustive, and might not reflect recent changes to the API. For current and canonical information about API operations and data structures, the [Swagger interface](./) is recommended.

## Operations on Configurations

Get the detailed list of existing configurations:

```
curl -XGET $api_url/configs/
```

Retrieve a complete configuration: 

```
curl -XGET $api_url/configs/{config}/
```

Create a new configuration, name is in the posted data:

```
curl -XPOST $api_url/configs/
```

Create a new configuration, name is provided and overrides posted data:

```
curl -XPOST $api_url/configs/{config}/
```

Update an existing configuration: 

```
curl -XPUT $api_url/configs/{config}/
```

Delete a configuration: 

```
curl -XDELETE $api_url/configs/{config}/
```

Clone a configuration, new name is in POST data:

```
curl -XPOST $api_url/configs/{config}/clone/
```

Clone a configuration, new name is in the URL: 

```
curl -XPOST $api_url/configs/{config}/clone/{new_name}/
```

Get all versions of a given configuration:

```
curl -XGET $api_url/configs/{config}/v/
```

Retrieve a specific version of a configuration:

```
curl -XGET $api_url/configs/{config}/v/{version}/
```

Create a new version for a configuration from an old version:

```
curl -XPUT $api_url/configs/{config}/v/{version}/revert/
```

#### Format for full config dumps and batch updates: <a href="markdown-header-format-for-full-config-dumps-and-batch-updates" id="markdown-header-format-for-full-config-dumps-and-batch-updates"></a>

```
{
    "config": { <fields of config metadata> }

    "resources": {
        "<resname>": [],
        ...
    }
    "blobs": {
        "dataname": {"format":"<format of blob, ex: raw, base64, gzip+base64, etc.>", 
                     "blob": "<blob>"
         }
    }
    "delete_resources": {
        "<resname>": {
            "<resid>": true,
            ...
        }
        ...
    }
    "delete_blobs": {
        "<blobname>": true,
        ...
    }
}
```

## Operations on Documents

Retrieve the list of existing documents in this configuration:

```
curl -XGET $api_url/configs/{config}/d/
```

Get a complete document:

```
curl -XGET $api_url/configs/{config}/d/{document}/
```

Get a given version of a document:

```
curl -XGET $api_url/configs/{config}/d/{document}/v/{version}/
```

Retrieve the existing versions of a given document:

```
curl -XGET $api_url/configs/{config}/d/{document}/v/
```

Create a new complete document:

```
curl -XPOST $api_url/configs/{config}/d/{document}/
```

Update an existing document:

```
curl -XPUT $api_url/configs/{config}/d/{document}/
```

Delete/empty a document:

```
curl -XDELETE $api_url/configs/{config}/d/{document}/
```

Create a new version for a document from an old version:

```
curl -XPUT $api_url/configs/{config}/d/{document}/v/{version}/revert/
```

## Operations on Entries

Retrieve the list of entries in a document:

```
curl -XGET $api_url/configs/{config}/d/{document}/e/
```

Retrieve an entry from a document:

```
curl -XGET $api_url/configs/{config}/d/{document}/e/{entry}/
```

Get the list of existing versions of a given entry in a document:

```
curl -XGET $api_url/configs/{config}/d/{document}/e/{entry}/v/
```

Get a given version of a document entry:

```
curl -XGET $api_url/configs/{config}/r/{document}/i/{entry}/v/{version}/
```

Create an entry in a document:

```
curl -XPOST $api_url/configs/{config}/d/{document}/e/
```

Update an entry in a document:

```
curl -XPUT $api_url/configs/{config}/d/{document}/e/{entry}/
```

Delete an entry from a document:

```
curl -XDELETE $api_url/configs/{config}/d/{document}/e/{entry}/
```

## Operations on Blobs

Retrieve the list of available blobs:

```
curl -XGET $api_url/configs/{config}/b/
```

Retrieve a blob:

```
curl -XGET $api_url/configs/{config}/b/{blob}/
```

Create a new blob:

```
curl -XPOST $api_url/configs/{config}/b/{blob}/
```

Replace a blob with new data:

```
curl -XPUT $api_url/configs/{config}/b/{blob}/
```

Delete a blob:

```
curl -XDELETE $api_url/configs/{config}/b/{blob}/
```

Retrieve the list of versions of a given blob:

```
curl -XGET $api_url/configs/{config}/b/{blob}/v/
```

Retrieve the given version of a blob:

```
curl -XGET $api_url/configs/{config}/b/{blob}/v/{version}/
```

Create a new version for a blob from an old version:

```
curl -XPUT $api_url/configs/{config}/b/{blob}/v/{version}/revert/
```
