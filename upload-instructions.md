# MyMiniFactory - Upload API

### Description

MyMiniFactory's Upload API provides HTTP endpoints to post a new object on MyMiniFactory with its metadata and upload the files (usually 3D files) associated to it.
This document goes through the steps of the upload process.

We stringly recommend using the API V2 with the full domain: `https://www.myminifactory.com/api/v2` followed by the API route.

### Authentication

In order to create an object and upload files on MyMiniFactory, a user should be authenticated. Please refer to the Authentication Service documentation. Once authenticated using Oauth2, all requests to the upload API endpoints should contain the Authorization header with the access token.

### Basic mechanism

The upload process consist of mostly two steps, the third step lets you check the status of the object at all time but is a good way to make sure that all the files have been successfully uploaded at the end of the process.

1. Send the information of the object you want to create with the list of files that will be attached
2. POST all the files with their upload_id to the file endpoint
3. (Optional) Check the status of the newly uploaded object to make sure all the files are successfully uploaded

### Supported file formats
We support a wide range of 3D file formats for the upload but currently only stl files are verified and available on to download on MyMiniFac. Pdf files can also be uploaded as instruction or documentation for the object.

List of supported formats for file upload:
`3dc, 3ds, ac, asc, blend, bvh, dae, dw, dwf, dxf, fbx, fbx, flt, gcode, geo, gra, iv, ive, kmz, lwo, lwz, mu, obj, osg, osgb, osgt, pdf, ply, scad, shp, stl, vpk, wrl, wrz, x, x3g`

The maximum per-file upload size is `100 MB`.

### Step 1: Create the object on MyMiniFactory

Post basic information to create an object and send the list of files that will be uploaded. The response will contain an object id and upload ids for each files requested to be uploaded.

#### Request
```
POST /api/v1/object HTTP/1.1
Authorization: Bearer 79f4bd9c-5fa0-4415-819a-eabe28163108
Content-Type: application/json; charset=utf-8
Content-Length: 192

{
  "name": "Well Designed Teapot",
  "description": "This Well designed teapot",
  "source_url": "https://example.com/teapot",
  "tags": "tea, teapot",
  "files": [
    {
      "filename": "teapot.stl",
      "bytes": "44784"
    }
  ]
}
```

#### Parameters
| parameter | description | required |
| --------- | ----------- | -------- |
| `name` | Object name | yes |
| `description` | Description of the object, can contain links | no |
| `source_url` | Source URL for the object (client application link) | no |
| `tags` | Comma separated list of words | no |
| `files` | An object must contain at least one file | yes |
| `filename` | Filename of the file that will be upload | yes |
| `bytes` | Size in Bytes of the file (< 100MB) | no |


#### Response
```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Host: www.myminifactory.com

{
  "id": 42,
  "name": "Well Designed Teapot",
  "object_status_url": "https://myminifactory.com/curation/status/42",
  "object_url": "https://myminifactory.com/object/well-designed-teapot-42",
  "files": [
    {
      "upload_id": "1589466a6a50a3",
      "filename": "teapot.stl"
    }
  ]
}
```

| parameter | description |
| --------- | ----------- |
| `id` | MyMiniFactory id of the object |
| `name` | Object name |
| `object_status_url` | MyMiniFactory URL to track the status and curation process of the object |
| `object_url` | MyMiniFactory URL of the object page |
| `files` | List of files to be uploaded |
| `upload_id` | Unique identifier to upload the file |
| `filename` | Filename of the file to be uploaded |


The `id` of the object and all the `upload_id` should be kept by the client for the next steps.
The response file array will return the files in the same order from the request.
An object on MyMiniFactory is verified test printed before being published, Therefore, the object pas will not be publicly available immediately and the publication status can be checked on the status page.


### Step 2: Post a file attached to the object

Using the given `upload_id`, post the post the actual file. This request must only contain the content of the file. Therefore, the `Content-Length` header the should be the size of the file.

#### Request
```
POST /api/v1/file?upload_id=1589466a6a50a3 HTTP/1.1
Authorization: Bearer 79f4bd9c-5fa0-4415-819a-eabe28163108
Host: www.myminifactory.com
Content-Type: application/octet-stream
Content-Length: 44784
Content-Disposition: filename="teapot.stl"

BINARY_FILE_DATA
```

#### Parameters
| parameter | description | required |
| --------- | ----------- | -------- |
| `upload_id` | Unique identifier to upload the file | yes |

#### Response
```
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Host: www.myminifactory.com

{
  "filename": "teapot.stl"
}
```

### Step 3 (optional): Check the status of the object

You can query this endpoint make sure the operation is finished and all the files have been correctly uploaded.
If a file is missing you can do Step 2 with its upload_id to upload it again.

#### Request
```
GET /api/v1/object/{id}/upload_status HTTP/1.1
Authorization: Bearer 79f4bd9c-5fa0-4415-819a-eabe28163108
Host: www.myminifactory.com
Content-Type: application/json; charset=utf-8
```

#### Response
```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Host: www.myminifactory.com

{
  "id": "42",
  "name": "Well Designed Teapot",
  "object_status_url": "https://myminifactory.com/curation/status/42",
  "object_url": "https://myminifactory.com/object/well-designed-teapot-42",
  "files": [
    {
      "upload_id": "1589466a6a50a3",
      "filename": "teapot.stl",
      "status": "uploaded"
    }
  ]
}
```

| parameter | description |
| --------- | ----------- |
| `id` | MyMiniFactory id of the object |
| `name` | Object name |
| `object_status_url` | MyMiniFactory URL to track the status and curation process of the object |
| `object_url` | MyMiniFactory URL of the object page |
| `files` | List to-be-uploaded files and their upload status |
| `upload_id` | Unique identifier to re-upload the file  |
| `filename` | Filename of the file |
| `status` | The different status values are  `pending`, `failed`, `done`|

If the status is `pending` or `failed` the file upload can be retried with step 2.

#### Error Response format

The API can return JSON errors with the relevant HTTP status code. An error response looks like this
```
HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Host: www.myminifactory.com

{
  "error": "access_denied",
  "error_description": "OAuth2 authentication required"
}
```
| parameter | description |
| --------- | ----------- |
| `error` | Error code string |
| `error_description` | Detailed description of the error |

That's it! If you have any issue with the upload process please contact us at info@myminifactory.com.