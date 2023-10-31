# DockerFile
機械学習フレームワークを構築するDockerfile

## Dockerfile_root
Dockerfile_rootは、rootless-Dockerに対応したDockerファイルです。
コンテナ内で作成したファイルの所有者情報のUIDとGIDをローカル環境のIDに一致させます。

## Dockerfile
通常のDockerファイルです。
Rootless等の指定がない場合、作成したファイルが管理者権限になることを防いでいます。