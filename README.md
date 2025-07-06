# Colima を使用した際の注意点をメモしていく
## docker login and docker push
`docker login`を行うと、~/.docker/config.json に認証情報が保存される。
しかし、Colima はこのファイルを参照しないため、docker login を行っても Colima 上の Docker には認証情報が反映されない。
よってイメージをpushするには以下の様にtagにdocker hubのユーザ名を指定する必要がある。
```bash
$ docker tag <your-image-name>:<tag> <your-dockerhub-username>/<your-image-name>:<tag>
$ docker push <your-dockerhub-username>/<your-image-name>:<tag>
```

## dockerのUpgrade
```bash
$ breaw upgrade colima docker docker-compose
```
`Server: Docker Engine`をupgradeするにはColimaを一旦deleteして再度作成する必要がある。
（Client: Docker Engineはbrewでdockerをupgradeしたら自動で反映される）

## buildkitをpluginとして導入
brewでインストールしたDockerはbuildkitが含まれてない。
よって以下の手順でbuildkitをpluginとして導入する。
- [ref1](https://zenn.dev/fastsnowy/articles/fd2920d4844bc9)
- [ref2](https://github.com/abiosoft/colima/discussions/273#discussioncomment-2684502)

```bash
$ ARCH=arm64 # 私はmacのm1なので'arm64'を指定
$ VERSION=v0.24.0 # versionのlink: https://github.com/docker/buildx/releases/

$ curl -SLO https://github.com/docker/buildx/releases/download/${VERSION}/buildx-${VERSION}.darwin-${ARCH}
$ mkdir -p ~/.docker/cli-plugins
$ mv buildx-${VERSION}.darwin-${ARCH} ~/.docker/cli-plugins/docker-buildx
$ chmod +x ~/.docker/cli-plugins/docker-buildx

# 確認
$ docker buildx version 
github.com/docker/buildx v0.24.0 d0e5e86c8b88ae4865040bc96917c338f4dd673c
```

## buildxをデフォルトで使用する
```bash
$ docker buildx install
```

## buildxの文字カラーを変更する
変更するには以下の様にdockerの先頭に`BUILDKIT_COLORS`を指定すれば良い。
```bash
BUILDKIT_COLORS="run=white:error=red:cancel=blue:warning=yellow" docker build ./
```

## docker compose commandの使用
Docker Compose v2以降は、`docker compose`コマンドを使用する。
colimaではHomebrewで`docker-compose`がインストールされている場合、`docker compose`コマンドが使用できない。以下の様なエラーが発生する。
```
zsh: command not found: docker-compose
```

以下のコマンドで解決策を確認できる。
```bash
$ brew info docker-compose
==> docker-compose: stable 2.38.1 (bottled), HEAD
Isolated development environments using Docker
https://docs.docker.com/compose/
Installed
/opt/homebrew/Cellar/docker-compose/2.36.2 (8 files, 60.3MB) *
  Poured from bottle using the formulae.brew.sh API on 2025-05-26 at 11:58:51
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/d/docker-compose.rb
License: Apache-2.0
==> Dependencies
Build: go ✘
==> Options
--HEAD
	Install HEAD version
==> Caveats
Compose is a Docker plugin. For Docker to find the plugin, add "cliPluginsExtraDirs" to ~/.docker/config.json:
  "cliPluginsExtraDirs": [
      "/opt/homebrew/lib/docker/cli-plugins"
  ]
==> Analytics
install: 37,650 (30 days), 98,795 (90 days), 341,597 (365 days)
install-on-request: 37,626 (30 days), 98,724 (90 days), 341,311 (365 days)
build-error: 55 (30 days)
```
この場合、以下の様に`~/.docker/config.json`に`cliPluginsExtraDirs`を追加することで解決できる。
```json
{
  "cliPluginsExtraDirs": [
    "/opt/homebrew/lib/docker/cli-plugins"
  ]
}
```
