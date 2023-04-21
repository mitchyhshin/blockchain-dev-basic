# **Section 2 - 프로젝트 생성** :gear:

# npm packages 설치

- 원하는 위치에 폴더 생성

- VSCode에서 File - Open Folder로 위에서 만든 폴더 선택

- node 프로젝트 시작
    ```
    npm init
    ```

- package.json에 라이브러리 추가하기

    ```json
    "devDependencies": {
        "@nomiclabs/hardhat-ethers": "^2.0.2",
        "@nomiclabs/hardhat-etherscan": "^2.1.3",
        "@nomiclabs/hardhat-waffle": "^2.0.1",
        "@openzeppelin/contracts": "^4.8.1",
        "@typechain/ethers-v5": "^5.0.0",
        "@types/chai": "^4.2.18",
        "@types/lodash": "^4.14.182",
        "@types/mocha": "^8.2.2",
        "@types/moment": "^2.13.0",
        "@typescript-eslint/eslint-plugin": "^5.49.0",
        "@typescript-eslint/parser": "^5.49.0",
        "chai": "^4.3.4",
        "dotenv": "^16.0.3",
        "eslint": "^8.32.0",
        "eslint-config-prettier": "^8.6.0",
        "eslint-plugin-import": "^2.27.5",
        "eslint-plugin-prettier": "^4.2.1",
        "ethereum-waffle": "^3.4.0",
        "ethers": "^5.7.2",
        "fs": "^0.0.1-security",
        "hardhat": "^2.12.5",
        "hardhat-typechain": "^0.3.5",
        "prettier": "^2.8.3",
        "prettier-plugin-solidity": "^1.1.1",
        "sol-merger": "^3.1.0",
        "ts-generator": "^0.1.1",
        "ts-node": "^10.9.1",
        "typechain": "^4.0.0",
        "typescript": "^4.9.4"
    }
    ```

- package 설치

    ```
    npm install
    ```

# hardhat.config.ts

- hardhat.config.ts 파일추가
    ```ts
    import { HardhatUserConfig } from "hardhat/types";

    const config: HardhatUserConfig = {
        solidity: {
            compilers: [
                {
                    version: "0.8.17",
                    settings: {
                        optimizer: {
                            enabled: true,
                            runs: 200,
                        },
                    },
                },
            ],
        },
        defaultNetwork: "hardhat",
        networks: {
            hardhat: {
                accounts: {
                    count: 10,
                },
            },
        },
        mocha: {
            timeout: 400000,
        },
    };

    export default config;

    ```

    - optimizer는 Contract 코드의 크기와 실행비용을 줄이기 위해 사용합니다. runs의 값은 코드크기와 실행비용을 절충하기 위한 값입니다. runs의 값이 클수록 Contract 코드의 크기는 커지고 실행비용은 줄어듭니다.

# VSCode Extentions 설치

- ESLint Extension 설치

- .eslintrc.js 파일 추가
    ```js
    module.exports = {
        env: {
            browser: false,
            node: true,
            es2021: true,
        },
        parser: '@typescript-eslint/parser',
        parserOptions: { project: './tsconfig.json' },
        plugins: ['@typescript-eslint'],
        extends: [
            'eslint:recommended',
            'prettier',
            'plugin:import/typescript',
            'plugin:@typescript-eslint/recommended',
        ],
        rules: {
            '@typescript-eslint/no-floating-promises': 'error',
            '@typescript-eslint/require-await': 'error',
            '@typescript-eslint/no-unused-vars': 'off',
            '@typescript-eslint/no-empty-function': 'off',
        },
    };
    ```
- .eslintignore 파일추가
    
    ```
    *.js
    ```

- Prettier Extension 설치

- .prettierrc.js 파일 추가
    ```js
    module.exports = {
        singleQuote: true,
        semi: true,
        useTabs: false,
        tabWidth: 2,
        trailingComma: 'all',
        printWidth: 80,
        arrowParens: 'avoid',
        overrides: [
            {
            files: '*.sol',
            options: {
                tabWidth: 4,
                singleQuote: false,
                printWidth: 120,
                explicitTypes: 'always',
            },
            },
        ],
    };
    ```

- Solidity Extension 설치

- .solhint.json 파일 추가
    ```json
    {
        "rules": {
            "no-unused-vars": "error",
            "const-name-snakecase": "error",
            "contract-name-camelcase": "error",
            "event-name-camelcase": "error",
            "func-name-mixedcase": "error",
            "func-param-name-mixedcase": "error",
            "modifier-name-mixedcase": "error",
            "private-vars-leading-underscore": "error",
            "var-name-mixedcase": "error",
            "imports-on-top": "error"
        }
    }

    ```

- /.vscode/settings.json 파일 추가

    ```json
    {
        "editor.codeActionsOnSave": {
            "source.fixAll.eslint": true
        },
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    }

    ```

- tsconfig.json 파일추가

    ```json
    {
        "compilerOptions": {
            /* Basic Options */
            // "incremental": true,                   /* Enable incremental compilation */
            "target": "es2017" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */,
            "module": "commonjs" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
            // "lib": [],                             /* Specify library files to be included in the compilation. */
            // "allowJs": true,                       /* Allow javascript files to be compiled. */
            // "checkJs": true,                       /* Report errors in .js files. */
            // "jsx": "react",                        /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
            "declaration": true /* Generates corresponding '.d.ts' file. */,
            "declarationMap": true /* Generates a sourcemap for each corresponding '.d.ts' file. */,
            "sourceMap": true /* Generates corresponding '.map' file. */,
            // "outFile": "./",                       /* Concatenate and emit output to single file. */
            "outDir": "./lib" /* Redirect output structure to the directory. */,
            // "rootDir": "./",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
            // "composite": true,                     /* Enable project compilation */
            // "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
            "removeComments": true /* Do not emit comments to output. */,
            // "noEmit": true,                        /* Do not emit outputs. */
            // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
            // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
            // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

            /* Strict Type-Checking Options */
            "strict": true /* Enable all strict type-checking options. */,
            "noImplicitAny": false /* Raise error on expressions and declarations with an implied 'any' type. */,
            "strictNullChecks": true /* Enable strict null checks. */,
            // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
            // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
            // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
            "noImplicitThis": true /* Raise error on 'this' expressions with an implied 'any' type. */,
            // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */

            /* Additional Checks */
            // "noUnusedLocals": true,                /* Report errors on unused locals. */
            // "noUnusedParameters": true,            /* Report errors on unused parameters. */
            // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
            // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */

            /* Module Resolution Options */
            // "moduleResolution": "node",            /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
            // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
            // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
            // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
            // "typeRoots": [],                       /* List of folders to include type definitions from. */
            // "types": [],                           /* Type declaration files to be included in compilation. */
            // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
            "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
            // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
            // "allowUmdGlobalAccess": true,          /* Allow accessing UMD globals from modules. */

            /* Source Map Options */
            // "sourceRoot": "",                      /* Specify the location where debugger should locate TypeScript files instead of source locations. */
            // "mapRoot": "",                         /* Specify the location where debugger should locate map files instead of generated locations. */
            // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
            // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

            /* Experimental Options */
            "experimentalDecorators": true /* Enables experimental support for ES7 decorators. */,
            // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */

            /* Advanced Options */
            "forceConsistentCasingInFileNames": true /* Disallow inconsistently-cased references to the same file. */,
            "resolveJsonModule": true
        },
        "exclude": ["lib", "old", "test-src", "node_modules", ".vscode"]
    }
    ```