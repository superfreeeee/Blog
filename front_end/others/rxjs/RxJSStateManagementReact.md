# RxJS 实战: 基于 BehaviorSubject 实现状态管理 & 结合 React

@[TOC](文章目录)

<!-- TOC -->

- [RxJS 实战: 基于 BehaviorSubject 实现状态管理 & 结合 React](#rxjs-实战-基于-behaviorsubject-实现状态管理--结合-react)
- [完整代码示例](#完整代码示例)
- [基于 BehaviorSubject 定义状态](#基于-behaviorsubject-定义状态)
- [自定义 Hook](#自定义-hook)
- [React-RxJS](#react-rxjs)
- [参考链接](#参考链接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/rxjs/rxjs_state_management_react](https://github.com/superfreeeee/Blog-code/tree/main/front_end/others/rxjs/rxjs_state_management_react)

# 基于 BehaviorSubject 定义状态

上一篇我们提到 RxJS 里面的 BehaviorSubject 非常适合作为一个响应式对象来帮助我们进行状态管理。今天我们来尝试手写一下基于 React 框架的封装

- `/src/store/state.ts`

一开始我们先来看我们可以利用 BehaviorSubject 做出哪些常用的状态

> 基础状态

```ts
export const nameSubject = new BehaviorSubject<string>('crystal');
export const ageSubject = new BehaviorSubject<number>(15);
```

> 导出量

也称为计算状态，类似 Vue 里面的 computed

```ts
export const maxAgeSubject = ageSubject.pipe(
  map((age) => (age >= 18 ? 18 : age)),
);
```

> 依赖多个状态的导出量

```ts
export const idCardSubject: Observable<IIDCard> = combineLatest({
  name: nameSubject,
  age: ageSubject,
});
```

使用 RxJS 的 `combineLatest`，会自动将两个 Observable 对象合并成一个新的 Observable

# 自定义 Hook

第一个当然少不了的就是 `useState`，几乎每个与 React 对接的外部状态管理都希望尽量贴合 React 原生 state 的写法

> useSubjectState(使用 useState 的函数签名)

```ts
export const useSubjectState = <T>(
  subject: Subject<T>,
  defaultValue: T,
): [T, (value: T) => void] => {
  const [state, setState] = useState(defaultValue);

  useEffect(() => {
    const subscription = subject.subscribe((value) => {
      setState(value);
    });
    return () => {
      subscription.unsubscribe();
    };
  }, [subject]);

  const setSubject = useCallback((value: T) => {
    subject.next(value);
  }, []);

  return [state, setSubject];
};
```

利用 useEffect 去 subscribe，然后将状态同步到该组件；setState 则是用 next 来实现，非常简单

下面我们也可以学习 Recoil 的语法，再进一步将状态与 setState 隔离开来，如此一来如果我们只需要 setState 的场景下，就不会因为 state 状态的改变而产生重渲染

> useSubjectValue(对照 recoil 的 useRecoilValue)

```ts
export const useSubjectValue = <T>(
  observable: Observable<T>,
  defaultValue: T,
): T => {
  const [state, setState] = useState(defaultValue);

  useEffect(() => {
    const subscription = observable.subscribe((value) => {
      setState(value);
    });
    return () => {
      subscription.unsubscribe();
    };
  }, [observable]);

  return state;
};
```

> useSetSubjectState(对照 recoil 的 useSetRecoilState)

```ts
export const useSetSubjectState = <T>(
  subject: Subject<T>,
): ((value: T) => void) => {
  const setSubject = useCallback((value: T) => {
    subject.next(value);
  }, []);

  return setSubject;
};
```

再者，我们可以进一步在调用之前进行绑定，自动生成与某一个 subject 相关的 Hooks

> createUseSubjectStateHooks：自动生成与指定 Subject 绑定的上述三种钩子

```ts
export const createUseSubjectStateHooks = <T>(
  subject: Subject<T>,
): UseSubjectStateHooks<T> => {
  const useSubjectValue = (defaultValue: T): T => {
    const [state, setState] = useState(defaultValue);

    useEffect(() => {
      const subscription = subject.subscribe((value) => {
        setState(value);
      });
      return () => {
        subscription.unsubscribe();
      };
    }, [subject]);

    return state;
  };

  const useSetSubjectState = (): ((value: T) => void) => {
    const setSubject = useCallback((value: T) => {
      subject.next(value);
    }, []);

    return setSubject;
  };

  const useSubjectState = (defaultValue: T): [T, (value: T) => void] => {
    const state = useSubjectValue(defaultValue);
    const setState = useSetSubjectState();
    return [state, setState];
  };

  return {
    useSubjectValue,
    useSetSubjectState,
    useSubjectState,
  };
};
```

除了直接使用状态之外，我们也可以构建一个专门监听 Subject 变化的回调函数

第一种你可以写成一个 state，然后再配合 useEffect 实现，但是这样一来每次 Subject 变化都会调用 setState 引发重渲染；然而我们只是想用来调用 callback，我们其实完全可以跳过状态的绑定，直接使用原生的 subject 来调用回调即可

> useSubjectSideEffect

```ts
export const useSubjectSideEffect = <T>(
  observable: Observable<T>,
  onSubjectChange: (value: T) => void,
) => {
  const cbRef = useRef(onSubjectChange);
  cbRef.current = onSubjectChange;

  useEffect(() => {
    const subscription = observable.subscribe((value) => {
      cbRef.current(value);
    });
    return () => {
      subscription.unsubscribe();
    };
  }, [observable]);
};
```

利用 useRef 固定 callback 的引用，我们就可以在 subscribe 的时候去调用最新的回调就可以了

# React-RxJS

实际上，RxJS 的作者们早就做了类似的工作，封装的还更好，将状态先从 Observable 利用 state 进行转换统一类型，然后再使用 `useStateObservable` API 接入状态

[传送门：React-RxJS](https://react-rxjs.org/)

# 参考链接

| Title                                                          | Link                                                                                                                                                                                       |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| RxJS                                                           | [https://rxjs.dev/](https://rxjs.dev/)                                                                                                                                                     |
| React-RxJS                                                     | [https://react-rxjs.org/](https://react-rxjs.org/)                                                                                                                                         |
| Recoil                                                         | [https://recoiljs.org/](https://recoiljs.org/)                                                                                                                                             |
| How to combine multiple rxjs BehaviourSubjects - stackoverflow | [https://stackoverflow.com/questions/42840891/how-to-combine-multiple-rxjs-behavioursubjects](https://stackoverflow.com/questions/42840891/how-to-combine-multiple-rxjs-behavioursubjects) |
