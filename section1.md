# **Section 1 - 환경설정** :gear:

# Visual Studio Code 소개및 설치

- 소개
    - Editor

    - 가벼움

    - Extension(Plugin)을 설치하여 다양한 언어로 개발 가능합니다.

    - 많은 개발자가 사용하고 있고 빠르게 기능 업데이트가 이루어지고 있습니다.

- Visual Studio Code 설치

    - Visual Studio Code 이하 줄여서 VSCode로 부릅니다

    - https://code.visualstudio.com/download or 구글에서 VSCode로 검색하여 다운로드 페이지로 이동

    - 본인의 OS에 맞는 VSCode를 다운로드 받아 설치, Mac에서는 Visual Studio Code.app을 다운로드 받고 Applications 폴더로 드래깅

    - 설치 중 아무런 옵션도 바꾸지 않고 그대로 설치 진행

    - VSCode 실행하여 에러가 발생하지 않으면 확인 끝

# Node.js 소개및 설치

- Node.js 소개

    - 비동기 이벤트 기반 Java Script(줄여서 JS라고 씀) 런타임입니다. 간단하게 설명하자면 JS파일을 실행 시켜주는 프로그램

    - ts-node 라이브러리를 사용하면 TS 파일도 Compile 없이 실행가능

    - Compile : 내가 작성한 소스 코드를 동등한 다른 형태의 무엇으로 변환하는 작업을 말합니다. 위에서 언급한 대로 TS(TypeScript) 파일에는 Compile 과정이 필요 없지만 Solidity 파일(Smart Contract)은 컴파일 과정이 필요합니다. Solidity 파일을 컴파일하면 어떤 파일이 만들어지고 어떻게 사용하는지는 이후 강의에서 알려드립니다.

- Node.js 설치

    - Node.js를 NodeJS 또는 Node라고 부르기도 함

    - https://nodejs.org/en/download/ or 구글에서 Nodejs로 검색하여 다운로드 페이지로 이동

    - 본인의 OS에 맞는 NodeJS를 다운로드 받아 설치, Mac에서는 node-vx.x.x.pkg 파일을 다운로드 받고 설치

    - 설치 중 아무런 옵션도 바꾸지 않고 그대로 설치 진행

    - 아래 코드를 VSCode Terminal 창에서 실행하여 version이 표시되면 확인 끝

        ```
        node --version
        ```

        ```
        npm --version
        ```

    - 이후 강의 진행부터는 OS와 상관없이 동일 합니다.
