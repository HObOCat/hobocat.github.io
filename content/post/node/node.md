---
title: "vue实现转存base64格式的文件下载"
aliases: 
tags: [Node]
date: 2024-07-04
time: 14:48
---

## 写一个工具类
- download.js
```js

/**
 * 下载Base64格式文件
 * @param {Base64} data 
 * @param {文件名} fileName 
 */
export const downloadFileByByte = (data, fileName) => {
  const blob = buildBlobByByte(data)
  downloadFile(blob, fileName)
}

/**
 * 将Base64格式文件转为 Blob
 * @param {Base64格式} data 
 * @returns blob
 */
export const buildBlobByByte = (data) => {
  const raw = window.atob(data)
  const rawLength = raw.length
  const uInt8Array = new Uint8Array(rawLength)
  for (let i = 0; i < rawLength; ++i) {
      uInt8Array[i] = raw.charCodeAt(i)
  }
  return new Blob([uInt8Array])
}



/**
 * 下载文件
 * @param {blob文件} blob 
 * @param {文件名} fileName 
 */
export const downloadFile = (blob, fileName) => {
  const link = document.createElement('a')
  link.href = window.URL.createObjectURL(blob)
  link.download = fileName
  // 此写法兼容可火狐浏览器
  document.body.appendChild(link)
  const evt = document.createEvent('MouseEvents')
  evt.initEvent('click', false, false)
  link.dispatchEvent(evt)
  document.body.removeChild(link)
}
```

## 调用

```js
  // 注意文件位置
  import { downloadFileByByte } from '@/utils/download'

  downloadFileByByte(base64Str, fileName)

```