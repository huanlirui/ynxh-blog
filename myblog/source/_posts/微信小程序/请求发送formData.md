小程序 wx.request 无法设置为 formdata 形式传参

解决方案：

1. 设置 header

```javascript
header: {
"Content-Type": "multipart/form-data;boundary=XXX"
},
```

2. 引入该函数

```javascript
export const getFormData = (obj = {}) => {
  let result = "";
  for (let name of Object.keys(obj)) {
    let value = obj[name];
    result += "\r\n--XXX" + '\r\nContent-Disposition: form-data; name="' + name + '"' + "\r\n" + "\r\n" + value;
  }
  return result + "\r\n--XXX--";
};
```

3. 使用时。把 data 传入 getFromData 处理后再调用 wx.requeset

```javascript
const _formData = getFormData({ xxxx });
```
