---
title: 【TypeScript】 `tsc -v` で `SyntaxError： Unexpected token '?'` が発生する。
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---


## 問題

- TypeScriptの開発環境を構築するために
- `apt install npm` -> `npm install -g typescript` -> `tsc -v` の順にコマンドを実行したとき
- 下記エラーが発生した。

```shell
(venv) home/foo# tsc -v
/usr/local/lib/node_modules/typescript/lib/_tsc.js:92
  for (let i = startIndex ?? 0; i < array.length; i++) {
                           ^

SyntaxError: Unexpected token '?'
    at wrapSafe (internal/modules/cjs/loader.js:915:16)
    at Module._compile (internal/modules/cjs/loader.js:963:27)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1027:10)
    at Module.load (internal/modules/cjs/loader.js:863:32)
    at Function.Module._load (internal/modules/cjs/loader.js:708:14)
    at Module.require (internal/modules/cjs/loader.js:887:19)
    at require (internal/modules/cjs/helpers.js:85:18)
    at Object.<anonymous> (/usr/local/lib/node_modules/typescript/lib/tsc.js:8:18)
    at Module._compile (internal/modules/cjs/loader.js:999:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1027:10)
```

## 原因

- バージョン指定せずにTypeScriptとNode.jsをインストールしたとき、TypeScript(v5.1x)とNode.js(v12.22.9)がインストールされる。
- TypeScript(v5.1)はNode.jsのバージョンが`v14.17`以上でないと動かないのでエラーになる。

> ES2020 and Node.js 14.17 as Minimum Runtime Requirements
> -- <cite>[TypeScript 5.1 > Breaking Changes](https://example.com)</cite>

## 解決策

インストールしたNode.js・npm・TypeScriptを削除して、バージョン`v14.17`以上のNode.jsをインストールする。

### 手順

- 1 TypeScript、npmをアンインストールする。
  - 1 `npm uninstall -g typescript ts-node`
  - 2 `apt remove npm`
- 2 nvmをインストールする。
  - nvmのURLは<https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating>を参照
  - 1 `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash`
  - 2 `ls -l ~/.nvm | grep ins*`
  - 3 `source ~/.bashrc`
  - 4 `vi ~/.bashrc`
    - .bashrcの最終行に次の２行が表示されていることを確認する。

        ```shell
        export NVM_DIR="$HOME/.nvm"
        [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
        [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
        ```

  - `echo $NVM_DIR`
  - `command -v nvm`を実行して、以下のようになればOK。

    ```shell
    (venv) home/foo# command -v nvm
    nvm
    ```

- 3 バージョン`v14.17`以上のNode.jsをインストールする。(今回はv18.17.0を使用)
  - `nvm install v18.17.0`
  - 正しくインストールできているか確認

    ```shell
    (venv) home/foo# node -v
    v18.17.0
    ```

- 4 npm、typescriptをインストール
  - `apt install npm`
  - 正しくインストールできているか確認

    ```shell
    (venv) home/foo# npm -v
    9.6.7
    ```

  - `npm install -g typescript ts-node`
  - 正しくインストールできているか確認

    ```shell
    (venv) home/foo# tsc -v
    Version 5.7.3
    (venv) home/foo# ts-node -v
    v10.9.2
    ```

## 参考URL

- TypeScript 5.1 > Breaking Changes
  - <https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-1.html#breaking-changes>
- "tsc" command showing, "SyntaxError: Unexpected token ?"
  - <https://stackoverflow.com/questions/76421238/tsc-command-showing-syntaxerror-unexpected-token>
- nvmを使ってNode.jsをインストールする
  - <https://qiita.com/pyon_kiti_jp/items/da5080e9c7454e935aeb>
