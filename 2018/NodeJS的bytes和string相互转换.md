stringToBytes:
``` javascript
function stringToBytes(str) {
    return [].slice.call(Buffer.from(str, 'utf8'));
}
```
如果浏览器端可使用new TextEncoder().encode(str)

bytesToString:
``` javascript
function bytesToString(bytes) {
    return Buffer.from(bytes).toString('utf8')
}
```
如果浏览器端可使用new TextDecoder().decode(bytes)