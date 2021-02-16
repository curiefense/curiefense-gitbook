# cURL Examples

Curiefense includes a command line tool: `curieconf_cli`.

In the discussion below, some examples will be given using the command line via this tool. Others will be shown for both `curieconf_cli` and `curl`. 

## Operations on Configurations

List all existing configurations:

```text
curieconf_cli conf list
```

Retrieve the full configuration named `master`:

```text
curieconf_cli conf get master
```

Retrieve all the different versions of the configuration named `master`:

```text
curieconf_cli conf list-versions master
```

Create a new configuration from a dump file:

```text
curieconf_cli conf get master > /tmp/dumpfile.json
# edit /tmp/dumpfile, change configuration id to a new name
curieconf_cli conf create  /tmp/dumpfile.json
curieconf_cli conf create -n overriden_conf_name < /tmp/dumpfile.json
```

Revert configuration `master` to version `3a370d4`:

```text
curieconf_cli conf revert master 3a370d4
```

Update a configuration from a batch update:

```text
curieconf_cli conf update master < /tmp/batch.update.json
```

Get the detailed list of existing configurations:

```text
CURL: curl -XGET $api_url/configs/ 
CLI:  curieconf_cli conf list
```

Retrieve a complete configuration: 

```text
CURL: -XGET $api_url/configs/{config}/ 
CLI:  curieconf_cli conf get {config}
```

Create a new configuration, name is in the posted data:

```text
CURL: curl -XPOST $api_url/configs/ 
CLI:  curieconf_cli conf create [{filename}]
```

Create a new configuration, name is provided and overrides posted data:

```text
 CURL: curl -XPOST $api_url/configs/{config}/ 
 CLI:  curieconf_cli conf create -n {config} [{filename}]
```

Update an existing configuration: 

```text
CURL: curl -XPUT $api_url/configs/{config}/ 
CLI:  curieconf_cli conf update {config} [{filename}]
```

Delete a configuration: 

```text
CURL: -XDELETE $api_url/configs/{config}/ 
CLI:  curieconf_cli conf delete {config}
```

Clone a configuration, new name is in POST data:

```text
 CURL: curl -XPOST $api_url/configs/{config}/clone/ 
 CLI:  CLI uses the other clone URL
```

Clone a configuration, new name is in the URL: 

```text
CURL: -XPOST $api_url/configs/{config}/clone/{new_name}/ 
CLI:  curieconf_cli conf clone {config} {newname}
```

Get all versions of a given configuration:

```text
CURL: curl -XGET $api_url/configs/{config}/v/ 
CLI:  curieconf_cli conf list-versions {config}
```

Retrieve a specific version of a configuration:

```text
CURL: curl -XGET $api_url/configs/{config}/v/{version}/ 
CLI:  curieconf_cli conf get {config} -v {version}
```

Create a new version for a configuration from an old version:

```text
CURL: curl -XPUT $api_url/configs/{config}/v/{version}/revert/ 
CLI:  curieconf_cli conf revert {config} {version}
```

#### Format for full config dumps and batch updates: <a id="markdown-header-format-for-full-config-dumps-and-batch-updates"></a>

```text
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

```text
CURL: curl -XGET $api_url/configs/{config}/d/ 
CLI:  curieconf_cli doc list {config}
```

Get a complete document:

```text
CURL: curl -XGET $api_url/configs/{config}/d/{document}/ 
CLI:  curieconf_cli doc get {config} {document}
```

Get a given version of a document:

```text
 CURL: curl -XGET $api_url/configs/{config}/d/{document}/v/{version}/ 
 CLI:  curieconf_cli doc get {config} {document} -v {version}
```

Retrieve the existing versions of a given document:

```text
CURL: curl -XGET $api_url/configs/{config}/d/{document}/v/ 
CLI:  curieconf_cli doc list-versions {config} {document}
```

Create a new complete document:

```text
CURL: curl -XPOST $api_url/configs/{config}/d/{document}/ 
CLI:  curieconf_cli doc create {config} {document} [{filename}]
```

Update an existing document:

```text
CURL: curl -XPUT $api_url/configs/{config}/d/{document}/ 
CLI:  curieconf_cli doc update {config} {document} [{filename}]
```

Delete/empty a document:

```text
CURL: curl -XDELETE $api_url/configs/{config}/d/{document}/ 
CLI:  curieconf_cli doc delete {config} {document}
```

Create a new version for a document from an old version:

```text
CURL: curl -XPUT $api_url/configs/{config}/d/{document}/v/{version}/revert/ 
CLI:  curieconf_cli doc revert {config} {document} {version}
```

## Operations on Entries

Retrieve the list of entries in a document:

```text
CURL: curl -XGET $api_url/configs/{config}/d/{document}/e/ 
CLI:  curieconf_cli entry list {config} {document}
```

Retrieve an entry from a document:

```text
CURL: curl -XGET $api_url/configs/{config}/d/{document}/e/{entry}/ 
CLI:  curieconf_cli entry get {config} {document} {entry}
```

Get the list of existing versions of a given entry in a document:

```text
 CURL: curl -XGET $api_url/configs/{config}/d/{document}/e/{entry}/v/ 
 CLI:  curieconf_cli entry list-versions {config} {document} {entry}
```

Get a given version of a document entry:

```text
CURL: curl -XGET $api_url/configs/{config}/r/{document}/i/{entry}/v/{version}/ 
CLI:  curieconf_cli entry get {config} {document} {entry} -v {version}
```

Create an entry in a document:

```text
CURL: curl -XPOST $api_url/configs/{config}/d/{document}/e/ 
CLI:  curieconf_cli entry create {config} {document} [{filename}]
```

Update an entry in a document:

```text
CURL: curl -XPUT $api_url/configs/{config}/d/{document}/e/{entry}/ 
CLI:  curieconf_cli entry update {config} {document} {entry} [{filename}]
```

Delete an entry from a document:

```text
CURL: curl -XDELETE $api_url/configs/{config}/d/{document}/e/{entry}/ 
CLI:  curieconf_cli entry delete {config} {document} {entry}
```

## Operations on Blobs

Retrieve the list of available blobs:

```text
CURL: curl -XGET $api_url/configs/{config}/b/ 
CLI: curieconf_cli blob list {config}
```

Retrieve a blob:

```text
CURL: curl -XGET $api_url/configs/{config}/b/{blob}/ 
CLI:  curieconf_cli blob get {config} {blob}
```

Create a new blob:

```text
CURL: curl -XPOST $api_url/configs/{config}/b/{blob}/ 
CLI:  curieconf_cli blob create {config} {blob} [{filename}]
```

Replace a blob with new data:

```text
CURL: curl -XPUT $api_url/configs/{config}/b/{blob}/ 
CLI:  curieconf_cli blob update {config} {blob} [{filename}]
```

Delete a blob:

```text
CURL: curl -XDELETE $api_url/configs/{config}/b/{blob}/ 
CLI:  curieconf_cli blob delete {config} {blob}
```

Retrieve the list of versions of a given blob:

```text
CURL: curl -XGET $api_url/configs/{config}/b/{blob}/v/ 
CLI:  curieconf_cli blob list-versions {config} {blob}
```

Retrieve the given version of a blob:

```text
CURL: curl -XGET $api_url/configs/{config}/b/{blob}/v/{version}/ 
CLI:  curieconf_cli blob get {config} {blob} -v {version}
```

Create a new version for a blob from an old version:

```text
 CURL: curl -XPUT $api_url/configs/{config}/b/{blob}/v/{version}/revert/ 
 CLI:  curieconf_cli blob revert {config} {blob} {version}
```

