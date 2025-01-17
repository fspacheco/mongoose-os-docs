# Blynk
| Github Repo | C Header | C source  | JS source |
| ----------- | -------- | --------  | ----------------- |
| [mongoose-os-libs/blynk](https://github.com/mongoose-os-libs/blynk) | [mgos_blynk.h](https://github.com/mongoose-os-libs/blynk/blob/master/include/mgos_blynk.h) | &nbsp;  | [api_blynk.js](https://github.com/mongoose-os-libs/blynk/blob/master/mjs_fs/api_blynk.js)         |



This Mongoose OS library allows your device remote control via
the Blynk platform. It currently only works with Blynk Legacy 
and is not compatible with Blynk IoT. Device side logic could be 
implemented in either C/C++ or JavaScript.

Blynk is a platform with iOS and Android apps to control Arduino,
Raspberry Pi and the likes over the Internet.

See example video at:

<iframe src="https://www.youtube.com/embed/9lTIN_WRWMs"
  width="560" height="315"  frameborder="0" allowfullscreen></iframe>

## How to use this library

In your Mongoose OS app, edit `mos.yml` file and add a reference to this
library. See an [example blynk app](https://github.com/mongoose-os-apps/blynk)
that does that.

## Device configuration

This library adds `blynk` configuration section to the device:

```bash
mos config-get blynk
{
  "auth": "YOUR_BLYNK_AUTH_TOKEN",
  "enable": true,
  "server": "blynk-cloud.com:8442"
}
```

In order for your device to authenticate with Blynk cloud, either use
Web UI to change the `blynk.auth` value, or in a terminal:

```bash
mos config-set blynk.auth=YOUR_BLYNK_AUTH_TOKEN
```



 ----- 

Blynk API.

This library supports only a subset of Blynk protocol - namely, virtual
pin reading and virtual pin writing. That is enough to implement a very
wide class of applications. It currently only works with Blynk Legacy and is not compatible with Blynk IoT.
 

 ----- 
#### (*blynk_handler_t)

```c
typedef void (*blynk_handler_t)(struct mg_connection *, const char *cmd,
                                int pin, int val, int id, void *user_data);
```
>  Blynk event handler signature. 
#### blynk_set_handler

```c
void blynk_set_handler(blynk_handler_t func, void *user_data);
```
>  Set Blynk event handler. Everytime a Blynk event is raised, func will be called to treat that Event.
Example:
```c
// Defines Default Handler Func, sets that func using blynk_set_handler. The handler Func checks if there is a Virtual Read command and if true, sends virtual write that sets s_read_virtual_pin to 1
void default_blynk_handler(struct mg_connection *c, const char *cmd,
                                  int pin, int val, int id, void *user_data) {
  if (strcmp(cmd, "vr") == 0) {
    if (pin == s_read_virtual_pin) {
      blynk_virtual_write(c, s_read_virtual_pin, 1, id);
    }
  } 
  (void) user_data;
}

enum mgos_app_init_result mgos_app_init(void) {
  blynk_set_handler(default_blynk_handler, NULL);
  return MGOS_APP_INIT_SUCCESS;
}
```

#### blynk_send
```c
void blynk_send(struct mg_connection *c, uint8_t type, uint16_t id,
                const void *data, uint16_t len);
```
>  Send data to the Blynk server. `c` is a network connection which is passed to the handler registered
with `blynk_set_handler`. `type` is one of the following:

- `BLYNK_RESPONSE`
- `BLYNK_LOGIN`
- `BLYNK_PING`
- `BLYNK_HARDWARE`

`id` is the internal blynk message id; if undefined it will be autogenerated. `data` holds a message to send. `len` holds the length of a message to send.
#### blynk_printf

```c
void blynk_printf(struct mg_connection *c, uint8_t type, uint16_t id,
                  const char *fmt, ...);
```
>  Same as as `blynk_send()`, formats message using `printf()` semantics. 
#### blynk_virtual_write

```c
void blynk_virtual_write(struct mg_connection *c, int pin, float val, int id);
```
>  Send a virtual write command. If id is undefined, it will be autogenerated.
This is a helper function that uses `blynk_send()`.

### JS API

 --- 
#### Blynk.send

```javascript
Blynk.send(conn, type, msg, id)
```
Send raw message to Blynk server.

`conn` is a network connection which is passed to the handler registered
with `Blynk.setHandler`. `type` is one of the following:

- `Blynk.TYPE_RESPONSE`
- `Blynk.TYPE_LOGIN`
- `Blynk.TYPE_PING`
- `Blynk.TYPE_HARDWARE`

`msg` is a string with the data to send. `id` is the internal blynk
message id; if undefined it will be autogenerated.

Return value: none.
Example:
```javascript
// Send "virtual write" command manually: write "1" to pin "16"
Blynk.send(conn, Blynk.TYPE_HARDWARE, 'vw\x0016\x001');
```
#### Blynk.virtualWrite

```javascript
Blynk.virtualWrite(conn, pin, val, id)
```
Write to the virtual pin. If id is undefined, it will be autogenerated.
This is a helper function that uses `Blynk.send()`.
Return value: none.
Example:
```javascript
// Send "virtual write" command: write "1" to pin "16"
Blynk.virtualWrite(conn, 16, 1);
```
#### Blynk.setHandler

```javascript
Blynk.setHandler(handler, userdata)
```
Set handler for the virtual pin reads / writes.

Example:
```javascript
Blynk.setHandler(function(conn, cmd, pin, val, id) {
  print(cmd, pin, val);
}, null);
```
