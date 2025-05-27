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
