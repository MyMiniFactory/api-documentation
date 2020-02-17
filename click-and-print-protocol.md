
# MyMiniFactory Click And Print Protocol

**Version**: `0.2.2`

**HTTPS Base URL**: `https://www.myminifactory.com`

## 1. Printer Registration

### **POST** `/api/v2/printer`

**Protocol**: `HTTPS`

**Headers**

```yaml
X-Api-Key: api_key
Content-Type: application/json
```

**Body type**

```JSON
{
  "manufacturer": "[string] Name of the manufacturer",
  "model": "[string] Printer model type",
  "firmware_version": "[string] Printer Firmware version",
  "serial_number": "[string] Unique serial number of the printer (avoid "." character)",
  "mac_address": "[string] (optional) Printer network card MAC address"
}
```

**Body example**

```JSON
{
  "manufacturer": "Printer Manufacturer Name",
  "model": "printer-model",
  "firmware_version": "1.0.0",
  "serial_number": "00ccd40e-72ca-4e79-a4b6-67c95e2e3f1c",
  "mac_address": "01:23:45:67:89:AB"
}
```

### Success Response Type

**Code**: `201 CREATED`

```JSON
{
  "code": "[string]",
  "printer_token": "[string] Token that the printer should save",
  "qr_url": "[string] Url that can be displayed as QR code",
  "qr_image_url": "[string] Url of the QR code in png format",
  "qr_image_url_bmp": "[string] Url of the QR code in bmp format"
}
```

<div style="page-break-after: always;"></div>

### Success Response Example

**Code**: `201 CREATED`

```JSON
{
  "code": "123456",
  "printer_token": "zddfq34dd54dfsd40",
  "qr_url": "https://mmf.io/app?c=123456&s=00ccd40e-72ca-4e79-a4b6-67c95e2e3f1c",
  "qr_image_url": "https://www.myminifactory.com/df34dfgd676df.png",
  "qr_image_url_bmp": "https://www.myminifactory.com/df34dfgd676df.bmp"
}
```

### Error Responses

**Code**: `401 Unauthorized`

```JSON
{
  "error": "access_denied",
  "error_description": "access_denied"
}
```

**Code**: `422 Unprocessable Entity`

```JSON
{
  "error": "unprocessable_entity",
  "error_description": "Model name missing"
}
```

<div style="page-break-after: always;"></div>

## 2. Printer Status Updates

This status is sent regularly (~ every 5 seconds) as long as the printer is connected to the internet.

**Direction**: `Printer -> Server`

**Protocol**: `MQTT`

**QoS**: `0`

**Host**: `mqtt.myminifactory.com`

**Ports**: `1883`, `8883 SSL`

**Username**: `client_id`

**Password**: `api_key`

**Topic**: `/printers`

#### Body Type

```JSON
{
  "action_code": "[integer] 300",
  "status": "[string] free | prepare | printing | done",
  "printer_token": "[string] Printer token for authentication",
  "manufacturer": "[string]",
  "model": "[string]",
  "firmware_version": "[string]",
  "serial_number": "[string]",
  "current_task_id": "[string] (optional) Current or last task id",
  "temperature": "[string] (optional) Hotend temperature in celsius degrees",
  "bed_temperature": "[string] (optional) Hotbed temperature in celsius degrees",
  "print_progress": "[integer] percentage",
  "remaining_time": "[integer] seconds remaining",
  "total_time": "[integer] total seconds to print",
  "date": "[string] current datetime in ISO 8601"
}
```

#### Body Example

```JSON
{
  "action_code": 300,
  "status": "free",
  "printer_token": "sdff34dd54dfsd45",
  "manufacturer": "Manufacturer Name",
  "model": "printer-model",
  "firmware_version": "1.0",
  "serial_number": "00ccd40e-72ca-4e79-a4b6-67c95e2e3f1c",
  "current_task_id": "a56ca-42sde79-a4b6",
  "temperature": "25",
  "bed_temperature": "25",
  "print_progress": 100,
  "remaining_time": 0,
  "total_time": 3600,
  "date": "2018-08-17T08:00:40Z"
}
```
<div style="page-break-after: always;"></div>

## 2.1 Printer Status Update Requests (optional)

If a printer seems to bt idle or offline, the server can invite a printer to send its status.
The printer should then respond by sending a status.

**Direction**: `Server -> Printer`

**Protocol**: `MQTT`

**QoS**: `0`

**Topic**: `/printers/{printer_token}`

```JSON
{
  "action_code": 300,
}
```

<div style="page-break-after: always;"></div>

## 3. Printer actions

The server can push action to a printer by sending MQTT messages.

Possible action codes:

- prepare: 100
- print 101
- pause: 102
- cancel: 103
- resume: 104

**Direction**: `Server -> Printer`

**Protocol**: `MQTT`

**QoS**: `0`

**Host**: `mqtt.myminifactory.com`

**Ports**: `1883`, `8883 SSL`

**Username**: `client_id`  <!--- Is authentication necessary for the printer to listen? -->

**Password**: `api_key`

**Topic**: `/printers/{printer_token}`

### **Body Example**: prepare

A print is about to start. The printer can start heating up and display the print process

```JSON
{
  "action_code": 100,
  "filename": "filename.gcode",
  "task_name": "123456-filename_bed0.gcode",
  "object_name": "Printable 3DBenchy",
  "designer_name": "Designer Name",
  "thumbnail_url_tiny": "https://cdn.myminifactory.com/[...]/70X70-[...].jpg",
  "thumbnail_url_small": "https://cdn.myminifactory.com/[...]/230X230-[...].jpg",
  "thumbnail_url_standard": "https://cdn.myminifactory.com/[...]/720X720-[...].jpg",
  "thumbnail_url_tiny_bmp": "https://cdn.myminifactory.com/[...]/70X70-[...].bmp",
  "thumbnail_url_small_bmp": "https://cdn.myminifactory.com/[...]/230X230-[...].bmp"
}
```

### **Body Example**: print

Print action: tells the printer to download the gcode and then start printing.
See 4. Download gcode file

```JSON
{
  "action_code": 101,
  "filename": "filename.gcode",
  "task_name": "123456-filename_bed0.gcode",
  "task_id": "72ca-4e79-a4b6",
  "object_name": "Printable 3DBenchy",
  "designer_name": "Designer Name",
  "thumbnail_url_tiny": "https://cdn.myminifactory.com/[...]/70X70-[...].jpg",
  "thumbnail_url_small": "https://cdn.myminifactory.com/[...]/230X230-[...].jpg",
  "thumbnail_url_standard": "https://cdn.myminifactory.com/[...]/720X720-[...].jpg",
  "thumbnail_url_tiny_bmp": "https://cdn.myminifactory.com/[...]/70X70-[...].bmp",
  "thumbnail_url_small_bmp": "https://cdn.myminifactory.com/[...]/230X230-[...].bmp"
}
```

### **Body Example**: pause

```JSON
{
  "action_code": 102,
  "task_id": "72ca-4e79-a4b6"
}
```

### **Body Example**: cancel

```JSON
{
  "action_code": 103,
  "task_id": "72ca-4e79-a4b6"
}
```

### **Body Example**: resume

```JSON
{
  "action_code": 104,
  "task_id": "72ca-4e79-a4b6"
}
```

<div style="page-break-after: always;"></div>

## 4. Download gcode file

### **GET** `/api/v2/print-file`

**Protocol**: `HTTPS`

#### Headers

```yaml
X-Api-Key: api_key
```

#### Query parameters

```yaml
task_id: [string] given task id
printer_token: [string] Saved printer token
```

#### Query parameters example
`?task_id=72ca-4e79-a4b6&printer_token=sdff34dd54dfsd45`

### Success Responses

**Code**: `200 OK`

```gcode
G17 G20 G90 G94 G54
G0 Z0.25
X-0.5 Y0
...
```

### Error Responses

**Code**: `401 Unauthorized`

```TEXT
Username/Password Authentication Failed.
```

**Code**: `403 Forbidden`

```JSON
{
  "error": "access_denied",
  "error_description": "access_denied"
}
```