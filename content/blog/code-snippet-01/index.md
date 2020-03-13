---
title: 代码片段一
date: "2020-03-13"
description: Some code.
---

* 十六进制字符串AES加解密

```
// crypto-js@3.x
import AES from 'crypto-js/aes';
import ECB from 'crypto-js/mode-ecb';
import pkcs7 from 'crypto-js/pad-pkcs7';
import Hex from 'crypto-js/enc-hex';
import Base64 from 'crypto-js/enc-base64';

const hexStr = 'yourhexstring';
const hex = Hex.parse(hexStr);
const key = 'yourkey'

const str = Base64.stringify(hex);
const decrypted = AES.decrypt(str, key, {
  mode: ECB,
  padding: pkcs7,
}).toString();

const encrypted = AES.encrypt(hex, key, {
  mode: ECB,
  padding: pkcs7,
}).ciphertext.toString();

```

* 十六进制字符串与ArrayBuffer之间转换

```
// ArrayBuffer->hex
function ab2hex(buffer) {
  var hexArr = Array.prototype.map.call(
    new Uint8Array(buffer),
    bit => {
      return ('00' + bit.toString(16)).slice(-2);
    }
  );
  return hexArr.join('');
}

// hex->ArrayBuffer
function hex2ab(hex) {
  var typedArray = new Uint8Array(
    hex.match(/[\da-f]{2}/gi).map(h => {
      return parseInt(h, 16);
    })
  );
  var buffer = typedArray.buffer;
  return buffer;
}

```

* 日期格式转换

```
// 把Date转化为指定格式
const formatDate = (date, format) => {
  const o = {
    'M+': date.getMonth() + 1,
    'd+': date.getDate(),
    'h+': date.getHours(),
    'm+': date.getMinutes(),
    's+': date.getSeconds(),
    'q+': Math.floor((date.getMonth() + 3) / 3),
    'S+': date.getMilliseconds(),
  };
  const week = {
    '0': '日',
    '1': '一',
    '2': '二',
    '3': '三',
    '4': '四',
    '5': '五',
    '6': '六',
  };
  if (/(y+)/.test(format)) {
    format = format.replace(RegExp.$1, `${date.getFullYear()}`.substr(4 - RegExp.$1.length));
  }
  if (/(E+)/.test(format)) {
    format = format.replace(
      RegExp.$1,
      (RegExp.$1.length > 1 ? (RegExp.$1.length > 2 ? '星期' : '周') : '') +
        week[date.getDay() + '']
    );
  }
  Object.keys(o).forEach(k => {
    if (new RegExp(`(${k})`).test(format)) {
      format = format.replace(
        RegExp.$1,
        RegExp.$1.length === 1 ? o[k] : `00${o[k]}`.substr(`${o[k]}`.length)
      );
    }
  });
  return format;
};
```

* Promise延时函数

```
const delay = timeout => {
  return new Promise(resolve => setTimeout(resolve, timeout));
}
```

胡说八道~