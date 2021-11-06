# JS: web 上的文件操作

@[TOC](文章目录)

<!-- TOC -->

- [JS: web 上的文件操作](#js-web-上的文件操作)
- [正文](#正文)
  - [1. web 上的文件操作](#1-web-上的文件操作)
  - [2. 文件来源](#2-文件来源)
    - [2.1 文件选择器](#21-文件选择器)
    - [2.2 拖拽文件](#22-拖拽文件)
  - [3. 提取文件内容](#3-提取文件内容)
    - [3.1 关于 FileReader](#31-关于-filereader)
    - [3.2 文件编辑](#32-文件编辑)
    - [3.3 文件展示](#33-文件展示)
      - [3.3.1 图片](#331-图片)
      - [3.3.2 视频](#332-视频)
      - [3.3.3 其他文件](#333-其他文件)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. web 上的文件操作

首先先明确 web 上如何操作文件，文件系统实际上是属于操作系统级别的较低层 API，实际上 web 浏览器能做的也就是访问系统上的 fs，获取相关文件的头信息或是查询内容罢了。

也就是说受限于浏览器的环境，我们在 web 应用中操作文件的方法是非常有限的。

## 2. 文件来源

web 上的几个文件的来源

- 本地文件
  - `<inpu type="file">` 触发文件选择器
  - 拖拽释放文件
- 非本地文件
  - 使用 XHR/fetch 方法获取远程文件

关于本地文件会得到一个 File 类型，类似于文件描述的文件，通常只会记录名字、类型、最后时间...之类的（细节自己查 API！[File - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/File)）

而要读内容则需要使用 `FileReader` 这个类，可以按多种格式分别读取实际内容；还可以直接使用 `URL.createObjectURL` 来创建对象引用 URL，这个等下再详细解说

参考：[FileReader - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)、[URL.createObjectURL() - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)

### 2.1 文件选择器

我们第一种获取文件的方式是使用 `<input type="file">` 的方式，然后按 change 事件从 files 属性上获取文件列表

- `/src/hooks/useFileSelector.tsx`

```ts
const onChange = useCallback(
  (e: ChangeEvent<HTMLInputElement>) => {
    const fileList = e.target.files;
    let newFiles = Array.from({ length: fileList.length }, (_, i) => fileList.item(i));
    newFiles = uniqueFiles([...files, ...newFiles]);
    setFiles(newFiles);
    onFilesChange && onFilesChange(newFiles);
  },
  [files, onFilesChange, options]
);
```

FileList 是一个比较特别的对象，只有 length、item 两种属性，转成 `File[]` 会好用一点

参考：[FileList - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList)

这时候我们可以从 File 对象获取一些基本属性

参考：[]()

- `/src/layouts/SelectFile.tsx`

```ts
<table>
  <thead>
    <tr>
      <th>文件名</th>
      <th>文件大小</th>
      <th>最后修改时间</th>
    </tr>
  </thead>
  <tbody>
    {files.map(({ name, size, lastModified }, i) => (
      <tr key={name}>
        <td>{name}</td>
        <td>{formatSize(size)}</td>
        <td>{new Date(lastModified).toLocaleString()}</td>
        <td>
          <DeleteButton onClick={() => removeFile(i)}>X</DeleteButton>
        </td>
      </tr>
    ))}
  </tbody>
</table>
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_1_input_1.png)

而这个确实是与本地上的文件相对应的

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_2_input_2.png)

### 2.2 拖拽文件

第二种我们还允许用户拖拽文件到页面并释放来进行上传

- `/src/hooks/useDragArea.tsx`

```ts
    const preventDefault = (e: SyntheticEvent) => {
      e.stopPropagation();
      e.preventDefault();
    };

    const selectFile = () => {
      inputRef.current.click();
    };

    const dragEnter = (e: DragEvent) => {
      preventDefault(e);

      !hover && setHover(true);
    };

    const dragLeave = (e: DragEvent) => {
      preventDefault(e);

      hover && setHover(false);
    };

    const dropFile = (e: DragEvent) => {
      hover && setHover(false);
      preventDefault(e);

      const files = e.dataTransfer.files;

      console.log('files', files);

      const file = files[0];
      if (file) {
        setFile(file);
      }
    };

    return (
      <>
        <InputFile ref={inputRef} onChange={onChange} />
        <DropArea
          className={classNames({ hover })}
          onClick={selectFile}
          onDragEnter={dragEnter}
          onDragOver={preventDefault}
          onDragLeave={dragLeave}
          onDrop={dropFile}
        />
      </>
    );
```

比较特别的是 hover 效果在拖拽的时候并不会生效，需要主动使用 dragEnter、dragLeave 来判断并设置样式

释放后在 `drop` 事件查询 `e.dataTransfer.files` 来获取文件列表，其他操作就与 input 元素的相似

- 效果

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_3_drag.png)

本地文件获取就到此，没了真的没了；关于远程文件先不解说，因为实际上远程文件将被转换成类似二进制流或 base64 编码的文件内容，本质上就是获取文件内容而不存在文件的概念

## 3. 提取文件内容

关于文件内容的提取，则要使用上面提过的 FileReader

### 3.1 关于 FileReader

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_4_filereader.png)

本质上就是能读取 File 类型（Blob 类型）对应的内容，并提供多种读取格式，会自动进行编码/转码等操作

### 3.2 文件编辑

大多数时候我们在 web 上对于文件的操作不算太多（用户上传文件、下载文件、展示文件内容）

关于更复杂如图片的处理、文件内容的修改，实际上仅仅是页面的 JS 操作，最终都是经过内容的编排，重新生成/上传文件来达成保存的目的

因此关于文件的详细操作我们可以拆分成几个步骤

- 获取文件内容
- 转换成可展示、可编辑的样式
- 前端收集用户修改，还原成文件对象(ObjectURL、Blob 等)
- 最终上传/下载文件来达成持久化的目的

然而本篇不过多阐述文件的编辑，仅仅给大家演示如何获取并展示文件内容啦

### 3.3 文件展示

我们分成几种文件类型进行展示

#### 3.3.1 图片

- `/src/layouts/FileViewer.tsx`

关于图片我们可以根据前面说的 `FileReader.readAsDataURL` 的形式，转换成 base64 编码后放到 `<img>` 元素上就行了

```ts
const reader = new FileReader();
reader.addEventListener('load', () => {
  viewerRef.current = <img src={reader.result as string} style={{ maxWidth: '100%' }} />;
  forceUpdate(); // 更新视图
});
reader.readAsDataURL(file);
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_5_image.png)

#### 3.3.2 视频

- `/src/layouts/FileViewer.tsx`

而对于视频文件我们则要使用 video 标签，这时候我们则可以使用 `URL.createObjectURL`，由浏览器创建指向目标文件对象的 URL 供 video 标签使用

```ts
const VideoViewer: FC<{ file: File }> = ({ file }) => {
  const videoRef = useRef<HTMLVideoElement>(null);

  const [objectUrl, setObjectUrl] = useState('');

  useEffect(() => {
    if (file) {
      const newUrl = window.URL.createObjectURL(file);
      setObjectUrl(newUrl);
    }
  }, [file]);

  return <video src={objectUrl} ref={videoRef} autoPlay controls />;
};
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_6_video.png)

#### 3.3.3 其他文件

而对于其他文件，我们一样可以透过 `URL.createObjectURL` 创建引用，然后直接在新页面或是 iframe 中打开

- `/src/layouts/FileViewer.tsx`

```ts
const link = document.createElement('a');

const objectUrl = window.URL.createObjectURL(file);
link.href = objectUrl;
link.target = '_blank';
link.click();
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/js_file_operation_7_pdf.png)

这里我们可以看到刚才贴上来的 `URL.createObjectURL` 的产物与 DataURL 是不同的，它类似于一个 Blob 文件引用，后面带上域名 + hashCode，的链接地址，可供不同场景使用

# 其他资源

## 参考连接

| Title                                    | Link                                                                                                                                                                               |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 在web应用程序中使用文件 - MDN            | [https://developer.mozilla.org/zh-CN/docs/Web/API/File/Using_files_from_web_applications](https://developer.mozilla.org/zh-CN/docs/Web/API/File/Using_files_from_web_applications) |
| \<input type="file"\> - MDN              | [https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input/file](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input/file)                                       |
| Blob - MDN                               | [https://developer.mozilla.org/zh-CN/docs/Web/API/Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)                                                                     |
| File - MDN                               | [https://developer.mozilla.org/zh-CN/docs/Web/API/File](https://developer.mozilla.org/zh-CN/docs/Web/API/File)                                                                     |
| FileList - MDN                           | [https://developer.mozilla.org/zh-CN/docs/Web/API/FileList](https://developer.mozilla.org/zh-CN/docs/Web/API/FileList)                                                             |
| DataTransfer - MDN                       | [https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer)                                                     |
| FileReader - MDN                         | [https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)                                                         |
| box-shadow - MDN                         | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow)                                                         |
| transition - MDN                         | [https://developer.mozilla.org/zh-CN/docs/Web/CSS/transition](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transition)                                                         |
| URL.createObjectURL() - MDN              | [https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)                                       |
| DOMString - MDN                          | [https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString)                                                           |
| 拖拽过程中，遇到“:hover“选择器的小坑     | [https://blog.csdn.net/m0_47955692/article/details/109257633](https://blog.csdn.net/m0_47955692/article/details/109257633)                                                         |
| 拖拽上传event.dataTransfer.files始终为空 | [https://blog.csdn.net/wang740209668/article/details/116862063](https://blog.csdn.net/wang740209668/article/details/116862063)                                                     |
| js实现控制文件拖拽并获取拖拽内容功能     | [https://www.jb51.net/article/135119.htm](https://www.jb51.net/article/135119.htm)                                                                                                 |
| 如何实现平滑的“box-shadow”动画效果       | [https://www.w3cplus.com/css3/how-to-animate-box-shadow.html](https://www.w3cplus.com/css3/how-to-animate-box-shadow.html)                                                         |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_file_operation](https://github.com/superfreeeee/Blog-code/tree/main/front_end/javascript/js_file_operation)
