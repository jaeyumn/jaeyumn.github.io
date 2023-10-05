---
title: "React @material-ui 설치 에러"
categories:
    - react
    - error
---


Material-UI는 React 기반의 프로젝트에서 미리 디자인된 UI 컴포넌트를 사용하여 개발자가 따로 컴포넌트나 CSS를 작성하지 않아도 된다.

그래서 프론트 공부 실습을 위해 해당 패키지를 설치하던 도중 에러가 발생했다.

### 에러 원인
---

```
PS C:\Users\wkdrn\..dev\todo\client> npm install @material-ui/core
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR! 
npm ERR! While resolving: client@0.1.0
npm ERR! Found: react@18.2.0
npm ERR! node_modules/react
npm ERR!   react@"^18.2.0" from the root project
npm ERR!
npm ERR! Could not resolve dependency:
npm ERR! peer react@"^16.8.0 || ^17.0.0" from @material-ui/core@4.12.4
npm ERR! node_modules/@material-ui/core
npm ERR!   @material-ui/core@"*" from the root project
npm ERR!
npm ERR! Fix the upstream dependency conflict, or retry
npm ERR! this command with --force or --legacy-peer-deps
npm ERR! to accept an incorrect (and potentially broken) dependency resolution.
npm ERR!
npm ERR!
npm ERR! For a full report see:
npm ERR! C:\Users\wkdrn\AppData\Local\npm-cache\_logs\2023-09-30T02_14_22_954Z-eresolve-report.txt

npm ERR! A complete log of this run can be found in: C:\Users\wkdrn\AppData\Local\npm-cache\_logs\2023-09-30T02_14_22_954Z-debug-0.log 
```

<br>

위 에러 메시지를 보면 React의 버전 충돌로 인해서 패키지 설치가 실패한 것이라고 나온다.

현재 프로젝트에서는 React 버전 '18.2.0'을 사용하고 있는데 Material-UI 패키지에서 요구하는 React 버전과 호환되지 않아서 발생되는 문제인 듯하다.

<br>

### 해결
---

위의 메시지에서는 여러 해결 방안을 제시해준다.

하나는 종속성 충돌을 피하기 위해 버전을 맞추어주거나, `--force` 또는 `--legacy-peer-deps` 명령어를 사용하여 강제로 설치하거나.

사실 `--force` 명령어를 추가해서 적용하는 것은 좋은 방법이라고 생각되지 않는다.

근본적 원인인 종속성 문제가 해결되지 않을 뿐더러 추후에 다른 문제가 발생할 수 있기 때문이다.

`--legacy-peer-deps` 명령어는 어떠한 알고리즘을 사용하여 종속성 충돌을 해결해준다고 한다. 나는 이 방식으로 문제를 해결했다.

다음에 동일한 문제가 발생하면 의존성 및 버전 관리에 주의하면서 문제를 해결해야겠다.

<br>

---

--force 옵션 명령어
```
npm install @material-ui/core @material-ui/icons --force
```

--legacy-peer-deps 옵션 명령어
```
npm install @material-ui/core @material-ui/icons --legacy-peer-deps
```