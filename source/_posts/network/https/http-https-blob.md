---
title: Https系列（三） Https+blob 视频播放
date: 2019-07-30 18:23:54
tags: [Https]
categories: [Https, 未完成]
description: https+blob 视频播放
---

## 简介

自从 HTML5 提供了 video 标签，在网页中播放视频已经变成一个非常简单的事，只要一个 video 标签 src 属性设置为视频的地址就完事了。由于 src 指向真实的视频网络地址，在早期一般网站资源文件不怎么**通过 referer 设置防盗链**，所以可以随意的下载视频，也就有了后来通过 blob 加密视频文件。

> 目前的云存储服务商大部分都支持 referer 防盗链。其原理就是在访问资源时，请求头会带上发起请求的页面地址，判断其不存在（表示直接访问图片地址）或不在白名单内，即为盗链。

现在许多视频在线观看网站，你如果打开`chrome`查看其`video`标签，会发现它的`src`是一个以`blob:`开头的地址。

## Blob 和 ArrayBuffer

`Blob`是**二进制原始数据**但是类似**文件**的对象，ArrayBuffer 对象用来表示**通用的、固定长度**的**原始二进制数据缓冲区**。
同时他们是可以互相转换的`Blob`和`ArrayBuffer`。

### Blob

`` Blob`对象表示一个`不可变、原始数据`的类文件对象。`Blob`表示的不一定是`JavaScript ``原生格式的数据。`File` 接口基于`Blob`，继承了 `blob` 的功能并将其扩展使其支持用户系统上的文件。
使用 `Blob()` 构造函穿新创建的 Blob 对象。常用的方法和属性如下：

- `slice()`方法： 创建包含另一个`blob`数据的子集`blob`
- `Blob.size`属性（只读）： Blob 对象中所包含数据的大小（字节）。
- `Blob.type`属性（只读）： 一个字符串，表明该 Blob 对象所包含数据的 MIME 类型。如果类型未知，则该值为空字符串。

> - 注意：slice()方法原本接受 length 作为第二个参数，以表示复制到新 Blob 对象的字节数。如果设置的参数使 start + length 超出了源 Blob 对象的大小，那返回的则是从 start 到结尾的数据。

- File 对象其实继承自 Blob 对象，并提供了提供了 name ， lastModifiedDate， size ，type 等基础元数据。

### ArrayBuffer

`ArrayBuffer`(length)来获得一片连续的内存空间，它不能直接读写，但可根据需要将其传递到 TypedArray 视图或 DataView 对象来解释原始缓冲区。实际上视图只是给你提供了一个某种类型的读写接口，让你可以操作`ArrayBuffer`里的数据。**TypedArray 需指定一个数组类型来保证数组成员都是同一个数据类型，而 DataView 数组成员可以是不同的数据类型**。

`TypedArray` 对象描述一个底层的**二进制数据缓存区**的一个类似数组(array-like)视图，`TypedArray`对象如下几种：

|     类型     | 大小（字节单位） |                 描述                 |    Web IDL type     | C 语言中的等效类型 |
| :----------: | :--------------: | :----------------------------------: | :-----------------: | :----------------: |
|  Int8Array   |        1         | 8 位二进制带符号整数 -2^7~(2^7) - 1  |        byte         |       int8_t       |
|  Uint8Array  |        1         |      8 位无符号整数 0~(2^8) - 1      |        octet        |      uint8_t       |
|  Int16Array  |        2         | 16 位二进制带符号整数 -2^15~(2^15)-1 |        short        |      int16_t       |
| Uint16Array  |        2         |     16 位无符号整数 0~(2^16) - 1     |   unsigned short    |      uint16_t      |
|  Int32Array  |        4         | 32 位二进制带符号整数 -2^31~(2^31)-1 |        long         |      int32_t       |
| Uint32Array  |        4         |     32 位无符号整数 0~(2^32) - 1     |    unsigned int     |      uint32_t      |
| Float32Array |        4         |          32 位 IEEE 浮点数           | unrestricted float  |       float        |
| Float64Array |        8         |          64 位 IEEE 浮点数           | unrestricted double |       double       |

## URL.createObjectURL

`URL.createObjectURL()` 静态方法会创建一个 `DOMString`，其中包含一个表示参数中给出的对象的`URL`。这个 `URL` 的**生命周期**和创建它的窗口中的 `document` 绑定。这个新的 URL 对象表示指定的 `File` 对象或 `Blob` 对象。

> `DOMString`可以说是会话**(session)**级的，所以你在新的`tab`打开也就无效了

video 标签的 src 属性，不管是相对路径，绝对路径，或者一个网络地址，归根结底都是指向一个`文件资源的地址`。上面的`Blob`其实是一个可以当作`文件`用的`二进制数据`，那么只要我们可以生成一个指向`Blob`的地址，再通过`URL.createObjectURL`生成一个临时地址，赋值给你`video`标签的`src`属性。

```javascript
const videoURL = URL.createObjectURL(video); // blob:https://www.aaa.com/12212aa1s1
```

通过`URL.revokeObjectURL(objectURL)` 释放一个之前已经存在的、通过调用 `URL.createObjectURL()` 创建的 `URL` 对象。

> 如果是以文件协议打开的 html 文件（即 url 为 file://开头），则地址中http://localhost:1234会变成null，而且此时这个Blob URL 是无法直接访问的。

## HLS 和 MPEG DASH

HLS （HTTP Live Streaming）, 是由 Apple 公司实现的基于 HTTP 的媒体流传输协议。HLS 以 ts 为传输格式，m3u8 为索引文件（文件中包含了所要用到的 ts 文件名称，时长等信息，可以用播放器播放，也可以用 vscode 之类的编辑器打开查看），在移动端大部分浏览器都支持，也就是说你可以用 video 标签直接加载一个 m3u8 文件播放视频或者直播，但是在 pc 端，除了苹果的 Safari，需要引入库来支持。
用到此方案的视频网站比如优酷，可以在视频播放时通过调试查看 Network 里的 xhr 请求，会发现一个 m3u8 文件，和每隔一段时间请求几个 ts 文件。
![https](../../images/http/https-1-1.png)
![https](../../images/http/https-1-2.png)
但是除了 HLS，还有 Adobe 的 HDS，微软的 MSS，方案一多就要有个标准点的东西，于是就有了 MPEG DASH。
DASH（Dynamic Adaptive Streaming over HTTP） ，是一种在互联网上传送动态码率的 Video Streaming 技术，类似于苹果的 HLS，DASH 会通过 media presentation description (MPD)将视频内容切片成一个很短的文件片段，每个切片都有多个不同的码率，DASH Client 可以根据网络的情况选择一个码率进行播放，支持在不同码率之间无缝切换。
Youtube，B 站都是用的这个方案。这个方案索引文件通常是 mpd 文件（类似 HLS 的 m3u8 文件功能），传输格式推荐的是 fmp4（Fragmented MP4）,文件扩展名通常为.m4s 或直接用.mp4。所以用调试查看 b 站视频播放时的网络请求，会发现每隔一段时间有几个 m4s 文件请求。
![https](../../images/http/https-1-3.png)
不管是 HLS 还是 DASH 们，都有对应的库甚至是高级的播放器方便我们使用，但我们其实是想要学习一点实现。其实抛开掉索引文件的解析拿到实际媒体文件的传输地址，摆在我们面前的只有一个如何将多个视频数据合并让 video 标签可以无缝播放。

> 与之相关的一篇 B 站文章推荐给感兴趣的朋友：[我们为什么使用 DASH](https://www.bilibili.com/read/cv855111/)

## MediaSource

video 标签 src 指向一个视频地址，视频播完了再将 src 修改为下一段的视频地址然后播放，这显然不符合我们无缝播放的要求。其实有了我们前面 Blob URL 的学习，我们可能就会想到一个思路，用 Blob URL 指向一个视频二进制数据，然后不断将下一段视频的二进制数据添加拼接进去。这样就可以在不影响播放的情况下，不断的更新视频内容并播放下去，想想是不是有点流的意思出来了。
要实现这个功能我们要通过 MediaSource 来实现，MediaSource 接口功能也很纯粹，作为一个媒体数据容器可以和 HTMLMediaElement 进行绑定。基本流程就是通过 URL.createObjectURL 创建容器的 BLob URL，设置到 video 标签的 src 上，在播放过程中，我们仍然可以通过 MediaSource.appendBuffer 方法往容器里添加数据，达到更新视频内容的目的。

```javascript
const video = document.querySelector('video');
//视频资源存放路径，假设下面有5个分段视频 video1.mp4 ~ video5.mp4，第一个段为初始化视频init.mp4
const assetURL = 'http://www.demo.com';
//视频格式和编码信息，主要为判断浏览器是否支持视频格式，但如果信息和视频不符可能会报错
const mimeCodec = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';
if ('MediaSource' in window && MediaSource.isTypeSupported(mimeCodec)) {
  const mediaSource = new MediaSource();
  video.src = URL.createObjectURL(mediaSource); //将video与MediaSource绑定，此处生成一个Blob URL
  mediaSource.addEventListener('sourceopen', sourceOpen); //可以理解为容器打开
} else {
  //浏览器不支持该视频格式
  console.error('Unsupported MIME type or codec: ', mimeCodec);
}

function sourceOpen() {
  const mediaSource = this;
  const sourceBuffer = mediaSource.addSourceBuffer(mimeCodec);
  let i = 1;
  function getNextVideo(url) {
    //ajax代码实现翻看上文，数据请求类型为arraybuffer
    ajax(url, function (buf) {
      //往容器中添加请求到的数据，不会影响当下的视频播放。
      sourceBuffer.appendBuffer(buf);
    });
  }
  //每次appendBuffer数据更新完之后就会触发
  sourceBuffer.addEventListener('updateend', function () {
    if (i === 1) {
      //第一个初始化视频加载完就开始播放
      video.play();
    }
    if (i < 6) {
      //一段视频加载完成后，请求下一段视频
      getNextVideo(`${assetURL}/video${i}.mp4`);
    }
    if (i === 6) {
      //全部视频片段加载完关闭容器
      mediaSource.endOfStream();
      URL.revokeObjectURL(video.src); //Blob URL已经使用并加载，不需要再次使用的话可以释放掉。
    }
    i++;
  });
  //加载初始视频
  getNextVideo(`${assetURL}/init.mp4`);
}
```

这段代码修改自 MDN 的 MediaSource 词条中的示例代码，原例子中只有加载一段视频，我修改为了多段视频，代码里面很多地方还可以优化精简，这里没做就当是为了方便我们看逻辑。
此时我们已经基本实现了一个简易的流媒体播放功能，如果愿意可以再加入 m3u8 或 mpd 文件的解析，设计一下 UI 界面，就可以实现一个流媒体播放器了。
最后提一下一个坑，很多人跑了 MDN 的 MediaSource 示例代码，可能会发现使用官方提供的视频是没问题的，但是用了自己的 mp4 视频就会报错，这是因为 fmp4 文件扩展名通常为.m4s 或直接用.mp4，但却是特殊的 mp4 文件。

## Fragmented MP4

通常我们使用的 mp4 文件是嵌套结构的，客户端必须要从头加载一个 MP4 文件，才能够完整播放，不能从中间一段开始播放。而 Fragmented MP4（简称 fmp4），就如它的名字碎片 mp4，是由一系列的片段组成，如果服务器支持 byte-range 请求，那么，这些片段可以独立的进行请求到客户端进行播放，而不需要加载整个文件。
我们可以通过这个网站判断一个 mp4 文件[是否为 Fragmented MP4 网站地址](http://nickdesaulniers.github.io/mp4info/)。
我们通过[FFmpeg](https://ffmpeg.org/)或[Bento4](https://www.bento4.com/)的 mp4fragment 来将普通 mp4 转换为 Fragmented MP4，两个工具都是命令行工具，按照各自系统下载下来对应的压缩包，解压后设置环境变量指向文件夹中的 bin 目录，就可以使用相关命令了。
Bento4 的 mp4fragment，没有太多参数，命令如下:

```shell
    mp4fragment video.mp4 video-fragmented.mp4
```

FFmpeg 会需要设置一些参数，命令如下：

```shell
    ffmpeg -i video.mp4 -movflags empty_moov+default_base_moof+frag_keyframe video-fragmented.mp4
```

> Tips：网上大部分的资料中转换时是不带 default_base_moof 这个参数的，虽然可以转换成功，但是经测试如果不添加此参数网页中 MediaSource 处理视频时会报错。
> 视频的切割分段可以使用 Bento4 的 mp4slipt，命令如下：

```shell
    mp4split video.mp4 --media-segment video-%llu.mp4 --pattern-parameters N
```

## 来个实例

服务端使用的`nodejs`，`koa`框架，这里的操作很简单，就是用`fs.readFileSync`直接打开视频文件，得到的`data`结果是二进制的数据，直接作为结果返回。

```javascript
const Koa = require('koa');
const Router = require('koa-router');
const buffer = require('buffer');
const app = new Koa();
const router = new Router();
const video = async (ctx, next) => {
  try {
    // open 一个放在服务器的视频
    let data = fs.readFileSync('XXX.XXX.XXX/simple.mp4');
    ctx.response.body = data;
  } catch (e) {
    return Promise.reject({
      status: 500,
      message: '视频传输错误'
    });
  }
  next();
};
router.get('/video', video);
app.use(router.routes()).use(router.allowedMethods());
app.liseten(3002);
```

前端代码，这里使用的最原生的`XMLHttpRequest`对象语法，这里最重要的一点是要设置`responseType 为 blob`，这样接收到`response`直接就是一个`blob`对象供我们使用。这个`responseType`属性不属于`http`头部信息，而是 ajax 请求中 XHR 对象的属性(默认为""也就是 text 类型，而在一些封装 XHR 的框架中，一般把默认值设为 json)。

```javascript
let xhr = new XMLHttpRequest();
xhr.open('GET', 'http://localhost:3001/video', true);
xhr.responseType = 'blob';
xhr.onload = function (e) {
  if (e.status === 200) {
    // 获取blob对象
    let blob = this.response;
    console.log(blob);
    // 获取blob对象地址，并把值赋给容器
    $('#sound').attr('src', URL.createObjectURL(blob));
  }
};
xhr.send();
```

这样就可以得到以 blob:开头的临时 url 地址，而且在向服务端请求时页隐藏了真实的视频地址.

## 引用

> [为什么视频网站的视频链接地址是 blob？](https://juejin.im/post/5d1ea7a8e51d454fd8057bea) > [通过 BLOB 加密视频文件](https://www.jianshu.com/p/04727924273d)
