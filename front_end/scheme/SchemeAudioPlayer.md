# 技术方案实践: 音频播放器封装

@[TOC](文章目录)

<!-- TOC -->

- [技术方案实践: 音频播放器封装](#技术方案实践-音频播放器封装)
- [前言](#前言)
- [正文](#正文)
  - [1. 播放器基础：\<audio\> 标签](#1-播放器基础audio-标签)
    - [1.1 重要属性](#11-重要属性)
    - [1.2 监听事件](#12-监听事件)
    - [1.3 可用方法](#13-可用方法)
  - [2. 音频播放器封装](#2-音频播放器封装)
    - [2.0 AudioPlayer 组件](#20-audioplayer-组件)
      - [2.0.1 使用](#201-使用)
      - [2.0.2 内部结构](#202-内部结构)
      - [2.0.3 封装核心钩子 useAudioControl](#203-封装核心钩子-useaudiocontrol)
    - [2.1 播放 / 暂停](#21-播放--暂停)
    - [2.2 播放时间 / 总时长](#22-播放时间--总时长)
    - [2.3 播放进度条拉杆](#23-播放进度条拉杆)
    - [2.4 音量控制](#24-音量控制)
    - [2.5 倍速控制](#25-倍速控制)
  - [3. 后续讨论](#3-后续讨论)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天给大家带来音频播放器使用和封装，使用的是 React 作为基础框架，当然在写法上其实也是能轻易的改造成框架无关的写法哦

# 正文

## 1. 播放器基础：\<audio\> 标签

在 web 也就是浏览器环境下药播放音频的基础在于 `<audio>` 标签的使用，在开始封装之前我们先来看看它主要有哪些属性、事件、可操作的方法等

### 1.1 重要属性

| 属性     | 含义                   | 示例                       |
| -------- | ---------------------- | -------------------------- |
| autoplay | 加载完毕后自动播放     | `<audio autoplay></audio>` |
| controls | 显示默认的音频控制栏位 | `<audio controls></audio>` |
| loop     | 循环播放               | `<audio loop></audio>`     |
| muted    | 是否静音               | `<audio muted></audio>`    |
| src      | 音频路径               | `<audio src=""></audio>`   |

这些属性我们也可以透过直接操作 Audio 对象来修改；上述这些主要是直接写在标签上的，下面这些则是操作 Audio 对象的时候会用到

| 属性         | 含义             | 只读? | 单位/类型   |
| ------------ | ---------------- | ----- | ----------- |
| currentSrc   | 当前音频 src     | ✅     |             |
| currentTime  | 当前播放时间     |       | number / 秒 |
| duration     | 时长             | ✅     | number / 秒 |
| muted        | 是否静音         |       | boolean     |
| playbackRate | 播放倍速         |       | number / 1  |
| seeking      | 当前是否正在拖拽 | ✅     | number / 1  |
| volume       | 音量             |       | number / 1  |

这里主要先列出几个接下来会用到的，除此之外还有几个也都是非常常见的可用属性

### 1.2 监听事件

除了一般属性之外，音频播放器最重要的莫过于事件了，音频的加载完成、失败、播放、状态的修改都必须透过事件来实现

| 事件         | 含义             | 示例                                               |
| ------------ | ---------------- | -------------------------------------------------- |
| playing      | 音频播放         | `audio.addEventListener('playing', listener)`      |
| pause        | 音频暂停         | `audio.addEventListener('pause', listener)`        |
| timeupdate   | 播放事件更新     | `audio.addEventListener('timeupdate', listener)`   |
| volumechange | 音量更新         | `audio.addEventListener('volumechange', listener)` |
| ratechange   | 播放倍速修改     | `audio.addEventListener('ratechange', listener)`   |
| seeking      | 拖拽播放时       | `audio.addEventListener('seeking', listener)`      |
| waiting      | 等待音频数据加载 | `audio.addEventListener('waiting', listener)`      |

### 1.3 可用方法

有了属性和事件之外，最重要的就是我们可以主动调用的 API，这也是操作音频的核心功能之一，当然我们很多操作也是透过主动修改属性来实现的

| 方法        | 含义           | 示例                      |
| ----------- | -------------- | ------------------------- |
| play        | 播放           | `audio.play()`            |
| pause       | 暂停           | `audio.pause()`           |
| load        | 重新加载       | `audio.load()`            |
| canPlayType | 检查可播放类型 | `audio.canPlayType(type)` |
| fastSeek    | 快速跳播       | `audio.fastSeek(time)`    |

但是像 Chrome 就没有提供 fastSeek 方法，我们则是必须透过直接设置 currentTime 的方式来实现相同的效果

## 2. 音频播放器封装

看完大部分的基础 API 之后，我们就可以来看看怎么样在 React 中封装一个音频播放器了

下面的实现看似强依赖 React，实际上我们可以轻易的解耦成一个 AudioPlayer 类型，并使用 `ref` 的方式重新嵌入到 React 应用当中

### 2.0 AudioPlayer 组件

首先先明确一下我们封装的最终目标：做出一个可复用的 AudioPlayer 音频播放组件

#### 2.0.1 使用

- `/fe/src/App.tsx`

```ts
const App: React.FC<{}> = () => {
  const [audioSrc, setAudioSrc] = useState('');

  useEffect(() => {
    const searchParams = new URLSearchParams(location.search);
    console.log(searchParams.get('n'));
    setAudioSrc(`http://localhost:8080/audio/${searchParams.get('n')}`);
  }, []);

  return (
    <div className={classNames(styles.app)}>
      <h1>React Template</h1>
      <AudioPlayer audioSrc={audioSrc} autoPlay />
    </div>
  );
};
```

根据查询参数决定音频路径，传入 AudioPlayer 组件即可

#### 2.0.2 内部结构

组件内部的结构大致如下

- `/fe/src/layouts/AudioPlayer/index.tsx`

```ts

interface IAudioPlayerProps {
  audioSrc: string;
  autoPlay?: boolean;
}

const AudioPlayer: FC<IAudioPlayerProps> = ({ audioSrc, autoPlay = false }) => {
  const audioRef = useRef<HTMLAudioElement>(null);

  useEffect(() => {
    console.log(audioRef.current);
    if (!audioRef.current) {
      console.log('audioRef.current empty');
      return;
    }

    audioRef.current.src = audioSrc;
  }, [audioSrc]);

  // audio 标签属性、事件封装
  const audioControl = useAudioControl(audioRef);
  const {
    state: { duration, playing, currentTime, volume, muted, speedRate, seekingRef },
    action: { play, pause, seek, updateCurrentTime, updateVolume, updateSpeedRate },
  } = audioControl;
  const realPercent = currentTime / duration;
  const speedOptions = [
    { value: 0.5 },
    { label: '正常', value: 1 },
    { value: 1.25 },
    { value: 1.5 },
    { value: 1.75 },
    { value: 2 },
  ];

  // 键盘事件封装
  useKeyboardEvent(audioControl);

  // 跳播
  const onTimePercentUpdate = useCallback(
    (percent: number, dragging: boolean) => {
      const currentTime = percent * duration;
      seekingRef.current = dragging;
      dragging ? updateCurrentTime(currentTime) : seek(currentTime);
    },
    [seek, duration, updateCurrentTime, seek]
  );

  const onVolumePercentUpdate = useCallback(
    (percent: number) => {
      updateVolume(percent);
    },
    [updateVolume]
  );

  const [currentSpeedIndex, setCurrentSpeedIndex] = useState(1);
  const switchSpeed = useCallback(() => {
    const nextOptionIndex = (currentSpeedIndex + 1) % speedOptions.length;
    setCurrentSpeedIndex(nextOptionIndex);
    const nextRate = speedOptions[nextOptionIndex].value;
    updateSpeedRate(nextRate);
  }, [currentSpeedIndex, speedOptions, updateSpeedRate]);

  return (
    //   组件 UI 部分
  );
};

export default AudioPlayer;
```

可以看出实际上我们封装的核心在于 useAudioControl 钩子的实现

#### 2.0.3 封装核心钩子 useAudioControl

- `/fe/src/layouts/AudioPlayer/hooks/useAudioControl.tsx`

```ts
export type TAudioControl = {
  state: {
    duration: number;
    playing: boolean;
    currentTime: number;
    volume: number;
    muted: boolean;
    speedRate: number;
    seekingRef: MutableRefObject<boolean>;
  };
  action: {
    play: () => void;
    pause: () => void;
    seek: (targetTime: number) => void;
    updateCurrentTime: (time: number) => void;
    updateVolume: (volume: number) => void;
    updateSpeedRate: (rate: number) => void;
  };
};

const useAudioControl = (audioRef: MutableRefObject<HTMLAudioElement>): TAudioControl => {
  // implementation

  return useMemo(
    () => ({
      state: {
        duration,
        playing,
        currentTime,
        volume,
        muted,
        speedRate,
        seekingRef,
      },
      action: {
        play,
        pause,
        seek,
        updateCurrentTime: setCurrentTime,
        updateVolume,
        updateSpeedRate,
      },
    }),
    [
      duration,
      playing,
      currentTime,
      volume,
      muted,
      speedRate,
      seekingRef,
      play,
      pause,
      seek,
      updateVolume,
      updateSpeedRate,
    ]
  );
};

export default useAudioControl;
```

接下来我们就可以来看看详细的封装实现了

### 2.1 播放 / 暂停

第一部分我们先来封装播放、暂停相关的属性和事件

- `/fe/src/layouts/AudioPlayer/hooks/useAudioControl.tsx`

```ts
const useAudioControl = (audioRef: MutableRefObject<HTMLAudioElement>): TAudioControl => {
  // 播放状态
  const [playing, setPlaying] = useState(false);
  
  useEffect(() => {
    if (!audioRef.current) {
      return;
    }

    const audio = audioRef.current;

    // 播放事件
    const onPlaying = () => {
      setPlaying(true);
    };
    // 暂停事件
    const onPause = () => {
      if (!seekingRef.current) {
        setPlaying(false);
      }
    };

    audio.addEventListener('playing', onPlaying);
    audio.addEventListener('pause', onPause);
    return () => {
      audio.removeEventListener('playing', onPlaying);
      audio.removeEventListener('pause', onPause);
    };
  }, []);

  // 播放
  const play = useCallback(() => {
    audioRef.current.play();
  }, [audioRef]);

  // 暂停
  const pause = useCallback(() => {
    audioRef.current.pause();
  }, [audioRef]);

  return useMemo(
    () => ({
      state: {
        playing,
      },
      action: {
        play,
        pause,
      },
    }),
    [
      // ...
    ]
  );
};

export default useAudioControl;
```

我们维护一个 playing 状态、然后向 audio 对象注册监听函数来维护这个播放状态

### 2.2 播放时间 / 总时长

- `/fe/src/layouts/AudioPlayer/hooks/useAudioControl.tsx`

第二个部分则是关于播放时间，关联的状态和事件为 `currentTime`、`timeupdate`；与总时长相关的则是  `duration`、`durationchange`

```ts
const useAudioControl = (audioRef: MutableRefObject<HTMLAudioElement>): TAudioControl => {
  // 音频时长
  const [duration, setDuration] = useState(0);
  // 播放时间
  const [currentTime, setCurrentTime] = useState(0);

  useEffect(() => {
    if (!audioRef.current) {
      return;
    }

    const audio = audioRef.current;

    const getDuration = (originDuration: number) => (isNaN(originDuration) ? 0 : Math.floor(originDuration));
    const getCurrentTime = (originCurrentTime: number) =>
      isNaN(originCurrentTime) ? 0 : Math.floor(originCurrentTime);

    // 初始化音量
    const { duration, currentTime, volume, muted, playbackRate } = audio;
    setDuration(getDuration(duration));
    setCurrentTime(getCurrentTime(currentTime));

    // 时间更新
    const onTimeUpdate = () => {
      if (!seekingRef.current) {
        setCurrentTime(getCurrentTime(audio.currentTime));
      }
    };
    // 时长改变
    const onDurationChange = () => {
      setDuration(getDuration(audio.duration));
    };

    audio.addEventListener('timeupdate', onTimeUpdate);
    audio.addEventListener('durationchange', onDurationChange);
    return () => {
      audio.removeEventListener('timeupdate', onTimeUpdate);
      audio.removeEventListener('durationchange', onDurationChange);
    };
  }, []);

  // 跳播
  const seek = useCallback(
    (targetTime: number) => {
      if (audioRef.current.fastSeek) {
        audioRef.current.fastSeek(targetTime);
      } else {
        audioRef.current.currentTime = targetTime;
      }
    },
    [audioRef]
  );

  return useMemo(
    () => ({
      state: {
        duration,
        currentTime,
      },
      action: {
        seek,
        updateCurrentTime: setCurrentTime,
      },
    }),
    [
      // ...
    ]
  );
};

export default useAudioControl;
```

### 2.3 播放进度条拉杆

对于播放器常见的拖拽查询播放时间的功能，我们则是透过一个 Slider 组件来实现

- `/fe/src/components/Slider/index.tsx`

```ts
const useAction = (
  bgRef: MutableRefObject<HTMLDivElement>,
  isVertical: boolean,
  onPercentUpdate?: (percent: number, seeking: boolean) => void
) => {
  const [manual, setManual] = useState(false);

  const bgRectRef = useRef<DOMRect>(null);

  const getBgRect = useCallback((): DOMRect => {
    const rect = bgRectRef.current;
    return rect || (bgRectRef.current = bgRef.current.getBoundingClientRect());
  }, []);

  const calcPercent = useCallback(
    (e: MouseEvent<HTMLDivElement>): number => {
      const { x, y, width, height } = getBgRect();
      const { clientX, clientY } = e;

      let percent: number;
      if (isVertical) {
        percent = (height - (clientY - y)) / height;
      } else {
        percent = (clientX - x) / width;
      }
      return Math.min(1, Math.max(0, percent));
    },
    [isVertical]
  );

  const handleClick = useCallback(
    (e: MouseEvent<HTMLDivElement>) => {
      onPercentUpdate && onPercentUpdate(calcPercent(e), false);
    },
    [onPercentUpdate, calcPercent]
  );

  const handleMouseDown = useCallback(
    (e: MouseEvent<HTMLDivElement>) => {
      onPercentUpdate && onPercentUpdate(calcPercent(e), true);
      setManual(true);
    },
    [onPercentUpdate, calcPercent]
  );

  const handleMouseMove = useCallback(
    (e: MouseEvent<HTMLDivElement>) => {
      if (manual) {
        onPercentUpdate && onPercentUpdate(calcPercent(e), true);
      }
    },
    [manual, onPercentUpdate, calcPercent]
  );

  const handleMouseUpOrLeave = useCallback(
    (e: MouseEvent<HTMLDivElement>) => {
      if (manual) {
        onPercentUpdate && onPercentUpdate(calcPercent(e), false);
        setManual(false);
      }
    },
    [manual, onPercentUpdate, calcPercent]
  );

  return {
    handleClick,
    handleMouseDown,
    handleMouseMove,
    handleMouseUpOrLeave,
  };
};
```

首先我们根据不同的用户行为来计算当前的比例

```ts
type TSliderDirection = 'vertical' | 'horizantal';

interface ISliderProps {
  direction?: TSliderDirection;
  percent?: number;
  onPercentUpdate?: (percent: number, seeking: boolean) => void;
}

const Slider: FC<ISliderProps> = ({ direction = 'horizantal', percent = 0, onPercentUpdate }) => {
  const isVertical = direction === 'vertical';
  const verticalStyle = { [styles.vertical]: isVertical };

  const bgRef = useRef<HTMLDivElement>(null);

  const { handleMouseDown, handleMouseMove, handleMouseUpOrLeave } = useAction(bgRef, isVertical, onPercentUpdate);
  const activePercent = percent * 100;

  return (
    <div
      className={classNames(styles.sliderBg, verticalStyle)}
      ref={bgRef}
      // onClick={handleClick}
      onMouseDown={handleMouseDown}
      onMouseMove={handleMouseMove}
      onMouseUp={handleMouseUpOrLeave}
      onMouseLeave={handleMouseUpOrLeave}
    >
      <div className={classNames(styles.slider, verticalStyle)}>
        <div
          className={classNames(styles.shadow, verticalStyle)}
          style={{ [isVertical ? 'height' : 'width']: `${activePercent}%` }}
        ></div>
        <div
          className={classNames(styles.dot, verticalStyle)}
          style={{ [isVertical ? 'bottom' : 'left']: `calc(-5px + ${activePercent}%)` }}
        ></div>
      </div>
    </div>
  );
};

export default Slider;
```

接下来提供一个拖拽条来与用户交互

最后我们在播放器组件里面利用 Slider 来展示和控制时间

- `/fe/src/layouts/AudioPlayer/index.tsx`

```ts
const AudioPlayer: FC<IAudioPlayerProps> = ({ audioSrc, autoPlay = false }) => {
  const realPercent = currentTime / duration;

  // 跳播
  const onTimePercentUpdate = useCallback(
    (percent: number, dragging: boolean) => {
      const currentTime = percent * duration;
      seekingRef.current = dragging;
      dragging ? updateCurrentTime(currentTime) : seek(currentTime);
    },
    [seek, duration, updateCurrentTime, seek]
  );

  return (
    //   ...
        <div className={styles.progressBar}>
          <Slider percent={realPercent} onPercentUpdate={onTimePercentUpdate} />
        </div>
    //   ...
  )
}
```

### 2.4 音量控制

音量控制的部分我们也可以通过 Slider 来实现，一样别忘记在 useAudioControl 加上相关属性和事件的管理

- `/fe/src/layouts/AudioPlayer/index.tsx`

```ts
const AudioPlayer: FC<IAudioPlayerProps> = ({ audioSrc, autoPlay = false }) => {
  const onVolumePercentUpdate = useCallback(
    (percent: number) => {
      updateVolume(percent);
    },
    [updateVolume]
  );

  return (
    <div className={styles.volumeControl}>
      <div className={styles.bar}>
        <Slider direction={'vertical'} percent={volume / 100} onPercentUpdate={onVolumePercentUpdate} />
      </div>
      <span>{`${volume}%`}</span>
    </div>
  )
}
```

- `/fe/src/layouts/AudioPlayer/hooks/useAudioControl.tsx`

```ts
const useAudioControl = (audioRef: MutableRefObject<HTMLAudioElement>): TAudioControl => {
  // 音量
  const [volume, setVolume] = useState(0);
  // 静音
  const [muted, setMuted] = useState(false);

  useEffect(() => {
    if (!audioRef.current) {
      return;
    }

    const audio = audioRef.current;
    const getVolumeRate = (originVolume: number) => (isNaN(originVolume) ? 0 : Math.floor(originVolume * 100));

    // 初始化音量
    const { duration, currentTime, volume, muted, playbackRate } = audio;
    setVolume(getVolumeRate(volume));
    setMuted(muted);

    // 音量更新
    const onVolumeChange = () => {
      const { volume, muted } = audio;
      setVolume(getVolumeRate(volume));
      setMuted(muted);
    };

    audio.addEventListener('volumechange', onVolumeChange);
    return () => {
      audio.removeEventListener('volumechange', onVolumeChange);
    };
  }, []);

  // 修改音量
  const updateVolume = useCallback(
    (volume: number) => {
      audioRef.current.volume = volume;
    },
    [audioRef]
  );

  return useMemo(
    () => ({
      state: {
        volume,
        muted,
      },
      action: {
        updateVolume,
      },
    }),
    [
        // ...
    ]
  );
};

export default useAudioControl;
```

### 2.5 倍速控制

倍速控制也是类似，不过我们这次采用列表的形式，限定用户只能在几个指定值做选择

- `/fe/src/layouts/AudioPlayer/hooks/useAudioControl.tsx`

```ts
const useAudioControl = (audioRef: MutableRefObject<HTMLAudioElement>): TAudioControl => {
  // 播放倍速
  const [speedRate, setSpeedRate] = useState(1);

  useEffect(() => {
    if (!audioRef.current) {
      return;
    }

    const audio = audioRef.current;
    // 初始化音量
    const { duration, currentTime, volume, muted, playbackRate } = audio;
    setSpeedRate(playbackRate);

    // 倍速改变
    const onRateChange = () => {
      setSpeedRate(audio.playbackRate);
    };

    audio.addEventListener('ratechange', onRateChange);
    return () => {
      audio.removeEventListener('ratechange', onRateChange);
    };
  }, []);

  // 修改倍速
  const updateSpeedRate = useCallback(
    (speedRate: number) => {
      audioRef.current.playbackRate = speedRate;
    },
    [audioRef]
  );

  return useMemo(
    () => ({
      state: {
        speedRate,
      },
      action: {
        updateSpeedRate,
      },
    }),
    [
        // ...
    ]
  );
};

export default useAudioControl;
```

- `/fe/src/layouts/AudioPlayer/index.tsx`

```ts
const AudioPlayer: FC<IAudioPlayerProps> = ({ audioSrc, autoPlay = false }) => {
  // ...

  // 倍速控制
  const speedOptions = [
    { value: 0.5 },
    { label: '正常', value: 1 },
    { value: 1.25 },
    { value: 1.5 },
    { value: 1.75 },
    { value: 2 },
  ];
  const [currentSpeedIndex, setCurrentSpeedIndex] = useState(1);
  const switchSpeed = useCallback(() => {
    const nextOptionIndex = (currentSpeedIndex + 1) % speedOptions.length;
    setCurrentSpeedIndex(nextOptionIndex);
    const nextRate = speedOptions[nextOptionIndex].value;
    updateSpeedRate(nextRate);
  }, [currentSpeedIndex, speedOptions, updateSpeedRate]);

  return (
    <div className={styles.speedOption} onClick={switchSpeed}>
      {(() => {
        const { label, value } = speedOptions[currentSpeedIndex];
        return label || `x${value}`;
      })()}
    </div>
    {speedOptions.map(({ label, value }, index) => (
      <div
        key={value}
        className={styles.speedOption}
        onClick={() => {
          setCurrentSpeedIndex(index);
          updateSpeedRate(value);
        }}
      >
        {label || `x${value}`}
      </div>
    ))}
  );
};
```

## 3. 后续讨论

本篇由于篇幅关系省略了很多其他也很重要的部分，大致可以分为以下几点

- 加载状态：在音频未加载完成时需要提供一个 loading 态
- 异常处理：音频加载异常、网络异常提示
- 已加载进度条：展示已经下载的音频资源长度
- 等待状态：播放直到加载资源不足时会进入等待状态

# 结语

本篇严格来说还是比较依赖 React API 的实现，后续作者可能希望能将这个音频播放器封装成一个复用性更好的完全框架无关的实现

# 其他资源

## 参考连接

| Title                                        | Link                                                                                                                                                                           |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| HTML \<audio\> 标签 - W3School               | [https://www.w3school.com.cn/tags/tag_audio.asp](https://www.w3school.com.cn/tags/tag_audio.asp)                                                                               |
| HTML DOM Audio 对象 - W3School               | [https://www.w3school.com.cn/jsref/dom_obj_audio.asp](https://www.w3school.com.cn/jsref/dom_obj_audio.asp)                                                                     |
| HTML 事件参考手册 - W3School                 | [https://www.w3school.com.cn/tags/html_ref_eventattributes.asp](https://www.w3school.com.cn/tags/html_ref_eventattributes.asp)                                                 |
| 基于react的audio组件                         | [https://blog.csdn.net/weixin_34032827/article/details/89128933](https://blog.csdn.net/weixin_34032827/article/details/89128933)                                               |
| js获取audio进度调播放状态 播放时间，详细     | [https://blog.csdn.net/qq_47703624/article/details/107556369](https://blog.csdn.net/qq_47703624/article/details/107556369)                                                     |
| Web 端使用 - iconfont                        | [https://www.iconfont.cn/help/detail](https://www.iconfont.cn/help/detail)                                                                                                     |
| HTML、CSS、JS对unicode字符的不同处理         | [https://www.cnblogs.com/liuxianan/p/display-unicode-character-in-html-css-and-js.html](https://www.cnblogs.com/liuxianan/p/display-unicode-character-in-html-css-and-js.html) |
| Element.getBoundingClientRect() - MDN        | [https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)               |
| 通过css可以使对应div标签内的文字换行或不换行 | [http://www.divcss5.com/rumen/r52776.shtml](http://www.divcss5.com/rumen/r52776.shtml)                                                                                         |
| KeyboardEvent.code - MDN                     | [https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent/code](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent/code)                                     |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_audio_player](https://github.com/superfreeeee/Blog-code/tree/main/front_end/scheme/scheme_audio_player)
