# JS 实战: 一文了解 5 种文件上传场景(React + Koa 实现)

@[TOC](文章目录)

<!-- TOC -->

- [JS 实战: 一文了解 5 种文件上传场景(React + Koa 实现)](#js-实战-一文了解-5-种文件上传场景react--koa-实现)
- [前言](#前言)
- [正文](#正文)
  - [1. 单文件上传](#1-单文件上传)
  - [2. 多文件上传](#2-多文件上传)
  - [3. 按目录多文件上传](#3-按目录多文件上传)
  - [4. 多文件合成压缩包上传](#4-多文件合成压缩包上传)
  - [5. 大文件分块上传](#5-大文件分块上传)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天来跟大家分享 5 个常见的文件上传场景，由浅入深，小心服用（示例代码仓库还另外附带使用 koa 实现的服务端负责保存代码，本篇就简单介绍前端部分的实现而已）

# 正文

本篇展示的前端代码使用了 React 作为载体，但是文件上传本身的能力是与 React 无关的

## 1. 单文件上传

单文件很简单，相信大家都多少知道一些

- `/fe/src/tests/Single.tsx`

主要核心的逻辑就是利用了 `<input type="file">` 标签来实现文件选择并上传

```ts
import React, { ChangeEvent, useState } from 'react'
import { group } from '../utils/msg'

interface UploadProps {
  url: string
  body: FormData
}

export const uploadREQ = ({ url, body }: UploadProps) => {
  return fetch(url, {
    method: 'POST',
    body,
  })
}

const Single = () => {
  const [filePath, setFilePath] = useState('')

  const onInputFileChange = (e: ChangeEvent<HTMLInputElement>) => {
    console.log('selected file:', e.target.files[0])
  }

  const upload = () => {
    const input = document.createElement('input')
    input.type = 'file'
    input.addEventListener('change', e => {
      const file = (e.target as HTMLInputElement).files[0]
      console.log('selected file:', file)

      const formData = new FormData()
      formData.append('file', file, `1_single_${file.name}`)

      const url = 'http://localhost:3001/upload/single'

      uploadREQ({ url, body: formData }).then(async res => {
        const result = await res.json()
        group(`[response] ${url}`, () => {
          console.log(result)
        })
        setFilePath(result.url)
      })
    })
    input.click()
  }

  return (
    <div>
      <h1>文件上传 - 1: 单文件上传</h1>
      <input
        id="input-file"
        type="file"
        onChange={onInputFileChange}
      />
      <input
        id="input-file2"
        type="file"
        accept=".png,.jpg"
        onChange={onInputFileChange}
      />
      <button id="btn-file3" onClick={upload}>
        Click to upload
      </button>
      <h4>
        file path:{' '}
        <a target="_blank" href={filePath}>
          {filePath}
        </a>
      </h4>
    </div>
  )
}

export default Single
```

撇去一些无关紧要的代码，我们可以看出就是以下几句为核心代码

- `e.target.fils` 或是 `inputElement.files` 获取选取文件

```ts
const file = (e.target as HTMLInputElement).files[0]
```

- 将文件包装到一个 FormData 中，然后作为 post 请求的 body 传给服务端就可以了

```ts
const formData = new FormData()
formData.append('file', file, `1_single_${file.name}`)

uploadREQ({ url, body: formData })
```

## 2. 多文件上传

- `/fe/src/tests/Multiple.tsx`

多文件其实也很简单，就是帮 `<input type="file" multiple>` 加上一个 multiple 的属性就可以选择多个文件了，其实本质上与单文件相同

```ts
import React, { ChangeEvent } from 'react'
import { group } from '../utils/msg'
import { uploadREQ } from './Single'

interface UploadFilesProps {
  url: string
  files: File[]
  prefix: string
  fromDir?: boolean
}

export const uploadFiles = ({
  url,
  files,
  prefix,
  fromDir = false,
}: UploadFilesProps) => {
  const formData = new FormData()

  files.forEach(file => {
    const fileName = fromDir
      ? // @ts-ignore
        file.webkitRelativePath.replace(/\//g, `@${prefix}_`)
      : `${prefix}_${file.name}`

    formData.append('files', file, fileName)
  })

  console.log('upload files:', formData.getAll('files'))

  return uploadREQ({ url, body: formData })
}

const Multiple = () => {
  const onFileChange = (e: ChangeEvent<HTMLInputElement>) => {
    const files = Array.from(e.target.files)

    const url = 'http://localhost:3001/upload/multiple'
    uploadFiles({
      url,
      files,
      prefix: '2_multiple',
    }).then(async res => {
      const result = await res.json()
      group(`[response] ${url}`, () => {
        console.log(result)
      })
    })
  }

  return (
    <div>
      <h1>文件上传 - 2: 多文件上传</h1>
      <input
        id="input-files"
        type="file"
        multiple
        onChange={onFileChange}
      />
    </div>
  )
}

export default Multiple
```

## 3. 按目录多文件上传

- `/fe/src/tests/Directory.tsx`

按目录上传比较特别的是使用 `webkitdirectory` 属性，虽然非标准属性，但是大多数浏览器都普遍支持

```ts
import React, { ChangeEvent } from 'react'
import { group } from '../utils/msg'
import { uploadFiles } from './Multiple'

const Directory = () => {
  const onFileChange = (e: ChangeEvent<HTMLInputElement>) => {
    const files = Array.from(e.target.files)

    const url = 'http://localhost:3001/upload/multiple'
    uploadFiles({
      url,
      files,
      prefix: '3_directory',
      fromDir: true,
    }).then(async res => {
      const result = await res.json()
      group(`[response] ${url}`, () => {
        console.log(result)
      })
    })
  }

  return (
    <div>
      <h1>文件上传 - 3: 按目录上传</h1>
      <input
        id="input-files"
        type="file"
        // @ts-ignore
        webkitdirectory="true"
        onChange={onFileChange}
      />
    </div>
  )
}

export default Directory
```

## 4. 多文件合成压缩包上传

- `/fe/src/tests/Zip.tsx`

第四种我们继承前面的多文件选择，不论是利用多文件还是目录上传，我们还要用另一个 `jszip` 这个包来生成压缩包，然后这个压缩包就可以作为单文件上传了

首先是压缩方法

```ts

const ZIP = (
  zipName: string,
  files: File[],
  options: JSZip.JSZipGeneratorOptions = {
    type: 'blob',
    compression: 'DEFLATE',
  }
): Promise<Blob> => {
  return new Promise((resolve, reject) => {
    const zip = new JSZip()
    files.forEach(file => {
      const path = (file as any).webkitRelativePath
      zip.file(path, file)
    })
    zip.generateAsync(options).then((bolb: Blob) => {
      resolve(bolb)
    })
  })
}
```

下面是页面核心代码

```ts

const Zip = () => {
  const onFileChange = async (e: ChangeEvent<HTMLInputElement>) => {
    const files = Array.from(e.target.files)

    // @ts-ignore
    const dirName = files[0].webkitRelativePath.split('/')[0]
    const zipName = `${dirName}.zip`
    const zipFile = await ZIP(zipName, files)

    const formData = new FormData()
    formData.append('file', zipFile, zipName)

    const url = 'http://localhost:3001/upload/single'
    uploadREQ({
      url,
      body: formData,
    }).then(async res => {
      const result = await res.json()
      group(`[response] ${url}`, () => {
        console.log(result)
      })
    })
  }

  return (
    <div>
      <h1>文件上传 - 4: 压缩文件上传</h1>
      <input
        id="input-files"
        type="file"
        // @ts-ignore
        webkitdirectory="true"
        onChange={onFileChange}
      />
    </div>
  )
}

export default Zip
```

## 5. 大文件分块上传

- `/fe/src/tests/BigFile.tsx`

最后一部分稍微比较复杂一些，我们一段一段的解释

要实现大文件上传的主要想法有几个要点

1. 生成文件特征值(MD5)并先向后端查询文件是否存在
2. 不存在时则开始将文件分块(chunk)并一一上传(多个块之间可以并发上传)
3. 最后在后端进行文件组合最后生成原文件

前端要做的事还是比较简单的，首先由于浏览器实际上会限制当前同域下的 http 并发请求数，所以我们可以自己实现一个并发请求管理

```ts
// 并发请求池
const asyncPool = async (
  poolLimit: number,
  tasks: any[],
  iteratorFn: (task: any, tasks?: any[]) => Promise<any>
) => {
  const waiting = [];
  const executing = [];
  for (const task of tasks) {
    // 创建异步任务
    const p = Promise.resolve().then(() => iteratorFn(task, tasks));
    waiting.push(p);

    // 任务数量超过池大小
    if (poolLimit <= tasks.length) {
      const e = p.then(() =>
        executing.splice(executing.indexOf(e), 1)
      );
      executing.push(e);

      if (executing.length >= poolLimit) {
        await Promise.race(executing);
      }
    }
  }
  return Promise.all(waiting);
};
```

第二个工具方法则是根据文件内容生成特征值，这边就要用上 spark-md5 这个库

```ts
import SparkMD5 from 'spark-md5';

// 计算文件 md5
const calcFileMD5 = (file: File): Promise<string> => {
  return new Promise((resolve, reject) => {
    const chunks = getChunks(file);
    let currentChunk = 0;
    const spark = new SparkMD5.ArrayBuffer();
    const fileReader = new FileReader();

    fileReader.onload = (e) => {
      spark.append(e.target.result as ArrayBuffer);
      currentChunk++;
      if (currentChunk < chunks) {
        loadNext();
      } else {
        resolve(spark.end());
      }
    };

    fileReader.onerror = (e) => {
      reject(fileReader.error);
      fileReader.abort();
    };

    function loadNext() {
      const start = currentChunk * chunkSize,
        end = Math.min(file.size, start + chunkSize);
      fileReader.readAsArrayBuffer(file.slice(start, end));
    }
    loadNext();
  });
};
```

接下来进入主要业务流程，第一个方法是向后端请求检查文件是否存在

```ts
interface ICheckFileExistRes {
  code: number;
  data: {
    isExists: boolean;
    [key: string]: any;
  };
}

// 检查文件是否存在
const checkFileExist = (
  name: string,
  md5: string,
  chunks: number
): Promise<ICheckFileExistRes> => {
  const params = qs.stringify({
    n: name,
    m: md5,
    c: chunks,
  });
  const url = `http://localhost:3001/upload/checkExist?${params}`;
  return fetch(url)
    .then((res) => res.json())
    .then((res) => {
      group(`[response] ${url}`, () => {
        console.log(res);
      });
      return res;
    });
};
```

第二个方法就是在文件不存在的时候分批上传

```ts
const chunkSize = 1024 * 1024; // 1MB

const getChunks = (file: File) => {
  return Math.ceil(file.size / chunkSize);
};

interface IUploadChunkProps {
  url: string;
  chunk: any;
  chunkId: number;
  chunks: number;
  fileName: string;
  fileMD5: string;
}

/**
 * 上传文件块
 * @param param0
 * @returns
 */
const uploadChunk = ({
  url,
  chunk,
  chunkId,
  chunks,
  fileName,
  fileMD5,
}: IUploadChunkProps) => {
  const formData = new FormData();
  formData.set('file', chunk, `${fileMD5}-${chunkId}`);
  formData.set('chunks', chunks + '');
  formData.set('name', fileName);
  formData.set('timestamp', Date.now().toString());

  return fetch(url, {
    method: 'POST',
    body: formData,
  }).then((res) => res.json());
};

interface IUploadFileProps {
  file: File;
  fileMD5: string;
  chunkIds: string[];
  chunkSize?: number;
  poolLimit?: number;
}

/**
 * 大文件上传
 */
const uploadFile = ({
  file,
  fileMD5,
  chunkIds,
  chunkSize = 1 * 1024 * 1024, // 1MB
  poolLimit = 3,
}: IUploadFileProps) => {
  const chunks = getChunks(file);
  return asyncPool(
    poolLimit,
    // @ts-ignore
    [...new Array(chunks).keys()],
    (i: number) => {
      if (chunkIds.includes(i + '')) {
        return Promise.resolve();
      }
      const start = i * chunkSize;
      const end = i + 1 === chunks ? file.size : start + chunkSize;
      const chunk = file.slice(start, end);
      return uploadChunk({
        url: 'http://localhost:3001/upload/chunk',
        chunk,
        chunkId: i,
        chunks,
        fileName: file.name,
        fileMD5,
      });
    }
  );
};
```

最后一个部分就是主要的组件代码

```ts
const BigFile = () => {
  const inputRef = useRef<HTMLInputElement>();

  const upload = async () => {
    // 获取文件基本信息
    const file = inputRef.current.files[0];
    const fileMD5 = await calcFileMD5(file);
    console.log('select file:', file);
    console.log('fileMD5:', fileMD5);

    // 检查文件是否存在
    const res = await checkFileExist(
      file.name,
      fileMD5,
      getChunks(file)
    );
    console.log('res', res);

    // 重新上传文件
    if (res.code && res.data.isExists) {
      console.log(`file exist: ${res.data.url}`);
    } else {
      const result = await uploadFile({
        file,
        fileMD5,
        chunkIds: res.data.chunkIds as string[],
      });
      console.log('result', result);
    }
  };

  const clear = () => {
    inputRef.current.value = '';
  };

  return (
    <div>
      <h1>文件上传 - 5: 大文件上传</h1>
      <input id="input-files" type="file" ref={inputRef} />
      <button onClick={upload}>Upload</button>
      <button onClick={clear}>Clear</button>
    </div>
  );
};

export default BigFile;
```

# 结语

其实概念上没什么比较困难的部分，有兴趣的同学可以到代码仓库里面看看实现，或是拉下来自己跑跑看

# 其他资源

## 参考连接

| Title                                                                                | Link                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 文件上传，搞懂这8种场景就够了                                                        | [https://mp.weixin.qq.com/s/uWqRhJc22iq_ek9cUbL3zQ](https://mp.weixin.qq.com/s/uWqRhJc22iq_ek9cUbL3zQ)                                                                                                                                                                     |
| Koa 官方文档                                                                         | [https://koa.bootcss.com/](https://koa.bootcss.com/)                                                                                                                                                                                                                       |
| 使用 Fetch - MDN                                                                     | [https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)                                                                                                                           |
| koajs/multer - Github                                                                | [https://github.com/koajs/multer](https://github.com/koajs/multer)                                                                                                                                                                                                         |
| 使用FormData上传多个文件                                                             | [https://blog.csdn.net/wang704987562/article/details/80304471](https://blog.csdn.net/wang704987562/article/details/80304471)                                                                                                                                               |
| FormData.append()                                                                    | [https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/append](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/append)                                                                                                                                       |
| express (using multer) Error: Multipart: Boundary not found, request sent by POSTMAN | [https://stackoverflow.com/questions/49692745/express-using-multer-error-multipart-boundary-not-found-request-sent-by-pos](https://stackoverflow.com/questions/49692745/express-using-multer-error-multipart-boundary-not-found-request-sent-by-pos)                       |
| Why isn't the FileList object an array?                                              | [https://stackoverflow.com/questions/25333488/why-isnt-the-filelist-object-an-array](https://stackoverflow.com/questions/25333488/why-isnt-the-filelist-object-an-array)                                                                                                   |
| Property does not exist on type 'DetailedHTMLProps, HTMLDivElement>' with React 16   | [https://stackoverflow.com/questions/46215614/property-does-not-exist-on-type-detailedhtmlprops-htmldivelement-with-react](https://stackoverflow.com/questions/46215614/property-does-not-exist-on-type-detailedhtmlprops-htmldivelement-with-react)                       |
| JSZip 官方文档                                                                       | [https://stuk.github.io/jszip/](https://stuk.github.io/jszip/)                                                                                                                                                                                                             |
| JavaScript 如何在线解压 ZIP 文件？                                                   | [https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247491236&idx=1&sn=80cc4629a22045f647bce87b6c28e083&scene=21](https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247491236&idx=1&sn=80cc4629a22045f647bce87b6c28e083&scene=21)                                 |
| 如何更好地理解中间件和洋葱模型                                                       | [https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247486886&idx=1&sn=63bffec358b77986558e868d1adc2183&scene=21](https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247486886&idx=1&sn=63bffec358b77986558e868d1adc2183&scene=21)                                 |
| JavaScript 中如何实现大文件并发上传？                                                | [https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247491853&idx=1&sn=aa59ee95df84a81f7b4700fa4fec9436&scene=21](https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247491853&idx=1&sn=aa59ee95df84a81f7b4700fa4fec9436&scene=21)                                 |
| satazor/js-spark-md5 - Github                                                        | [https://github.com/satazor/js-spark-md5](https://github.com/satazor/js-spark-md5)                                                                                                                                                                                         |
| JavaScript 中如何实现并发控制？                                                      | [https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247490704&idx=1&sn=18976b9c9fe2456172c394f1d9cae88b&scene=21#wechat_redirect](https://mp.weixin.qq.com/s?__biz=MzI2MjcxNTQ0Nw==&mid=2247490704&idx=1&sn=18976b9c9fe2456172c394f1d9cae88b&scene=21#wechat_redirect) |
| NPM酷库：qs，解析URL查询字符串                                                       | [https://segmentfault.com/a/1190000012874916](https://segmentfault.com/a/1190000012874916)                                                                                                                                                                                 |
| ljharb/qs - Github                                                                   | [https://github.com/ljharb/qs](https://github.com/ljharb/qs)                                                                                                                                                                                                               |
| 网址URL中特殊字符转义编码                                                            | [https://blog.csdn.net/pcyph/article/details/45010609](https://blog.csdn.net/pcyph/article/details/45010609)                                                                                                                                                               |
| JavaScript 中如何实现大文件并发上传？ - Github issue                                 | [https://gist.github.com/semlinker/b211c0b148ac9be0ac286b387757e692](https://gist.github.com/semlinker/b211c0b148ac9be0ac286b387757e692)                                                                                                                                   |
| koajs/koa-body - Github                                                              | [https://github.com/koajs/koa-body](https://github.com/koajs/koa-body)                                                                                                                                                                                                     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_upload](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_upload)
