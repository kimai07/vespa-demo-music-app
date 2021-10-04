# vespa-demo-music-app

## vespa-cli

インストール

```bash
$ brew install vespa-cli
```

バージョン確認

```bash
$ vespa version
vespa version 7.469.18 compiled with go1.17.1 on darwin/amd64
```

## vespa clone

雛形アプリ確認

```bash
$ vespa clone -l
album-recommendation
album-recommendation-monitoring
:
vespa-cloud/vespa-documentation-search
```

album-recommendationをmy-appという名前でクローン

```bash
$ vespa clone album-recommendation my-app
Downloading sample apps ...
Created my-app
$ cd my-app
$ tree
.
├── README.md
└── src
    ├── main
    │   └── application
    │       ├── hosts.xml
    │       ├── schemas
    │       │   └── music.sd
    │       ├── search
    │       │   └── query-profiles
    │       │       ├── default.xml
    │       │       └── types
    │       │           └── root.xml
    │       └── services.xml
    └── test
        └── resources
            ├── A-Head-Full-of-Dreams.json
            ├── Hardwired...To-Self-Destruct.json
            ├── Liebe-ist-fur-alle-da.json
            ├── Love-Is-Here-To-Stay.json
            └── When-We-All-Fall-Asleep-Where-Do-We-Go.json

9 directories, 11 files
```

## dockerコンテナ起動

起動

```bash
$ docker-compose up -d
```

起動時のコンテナログ確認

```bash
$ docker logs -f b89efb35df9b
Running /opt/vespa/libexec/vespa/start-configserver
Creating data directory /opt/vespa/var/zookeeper/version-2
runserver(configserver) running with pid: 275
Running /opt/vespa/libexec/vespa/start-vespa-base.sh
Starting config proxy using tcp/vespa-container:19070 as config source(s)
runserver(configproxy) running with pid: 575
Waiting for config proxy to start
config proxy started after 1s (runserver pid 575)
runserver(config-sentinel) running with pid: 671
```

## deploy

アプリケーションのデプロイ

```bash
$ vespa deploy ./my-app --wait 300
Success: Deployed my-app/src/main/application

Waiting up to 300 seconds for services to become available ...
Waiting up to 300 seconds for service to become ready ...
Container (query API) at http://127.0.0.1:8080 is ready
```

## ステータスチェック

Deploy API

```bash
$ vespa status deploy --wait 300
Waiting up to 300 seconds for services to become available ...
Waiting up to 300 seconds for service to become ready ...
Deploy API at http://127.0.0.1:19071 is ready
```

Document API

```bash
$ vespa status document
Container (document API) at http://127.0.0.1:8080 is ready
```

Query API

```bash
$ vespa status query
Container (query API) at http://127.0.0.1:8080 is ready
```

- 以下でも確認可能

```bash
$ vespa status
Container (query API) at http://127.0.0.1:8080 is ready
```


## データ投入

```bash
$ cd my-app
$ vespa document src/test/resources/A-Head-Full-of-Dreams.json
Success: put id:mynamespace:music::a-head-full-of-dreams
$ vespa document src/test/resources/Love-Is-Here-To-Stay.json
Success: put id:mynamespace:music::love-id-here-to-stay
$ vespa document src/test/resources/Hardwired...To-Self-Destruct.json
Success: put id:mynamespace:music::hardwired-to-self-destruct
```

## 検索

musicインデックスのalbumフィールドに 'head' を含むドキュメントを検索する

```bash
$ vespa query "select * from music where album contains 'head';"
{
    "root": {
        "id": "toplevel",
        "relevance": 1.0,
        "fields": {
            "totalCount": 1
        },
        "coverage": {
            "coverage": 100,
            "documents": 3,
            "full": true,
            "nodes": 1,
            "results": 1,
            "resultsFull": 1
        },
        "children": [
            {
                "id": "id:mynamespace:music::a-head-full-of-dreams",
                "relevance": 0.16343879032006284,
                "source": "music",
                "fields": {
                    "sddocname": "music",
                    "documentid": "id:mynamespace:music::a-head-full-of-dreams",
                    "artist": "Coldplay",
                    "album": "A Head Full of Dreams",
                    "year": 2015,
                    "category_scores": {
                        "cells": [
                            {
                                "address": {
                                    "cat": "pop"
                                },
                                "value": 1.0
                            },
                            {
                                "address": {
                                    "cat": "rock"
                                },
                                "value": 0.20000000298023224
                            },
                            {
                                "address": {
                                    "cat": "jazz"
                                },
                                "value": 0.0
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

sddocnameフィールドにmusicを含むドキュメントを検索する

- catの値に応じたweightが設定されスコア計算される

cat|weight
---|---
pop|0.8
rock|0.2
jazz|0.1

```bash
$ vespa query "select * from sources * where sddocname contains 'music';" "ranking=rank_albums" "ranking.features.query(user_profile)={{cat:pop}:0.8,{cat:rock}:0.2,{cat:jazz}:0.1}"
{
    "root": {
        "id": "toplevel",
        "relevance": 1.0,
        "fields": {
            "totalCount": 3
        },
        "coverage": {
            "coverage": 100,
            "documents": 3,
            "full": true,
            "nodes": 1,
            "results": 1,
            "resultsFull": 1
        },
        "children": [
            {
                "id": "id:mynamespace:music::a-head-full-of-dreams",
                "relevance": 0.8400000147521496,
                "source": "music",
                "fields": {
                    "sddocname": "music",
                    "documentid": "id:mynamespace:music::a-head-full-of-dreams",
                    "artist": "Coldplay",
                    "album": "A Head Full of Dreams",
                    "year": 2015,
                    "category_scores": {
                        "cells": [
                            {
                                "address": {
                                    "cat": "pop"
                                },
                                "value": 1.0
                            },
                            {
                                "address": {
                                    "cat": "rock"
                                },
                                "value": 0.20000000298023224
                            },
                            {
                                "address": {
                                    "cat": "jazz"
                                },
                                "value": 0.0
                            }
                        ]
                    }
                }
            },
            {
                "id": "id:mynamespace:music::love-id-here-to-stay",
                "relevance": 0.40000002831220627,
                "source": "music",
                "fields": {
                    "sddocname": "music",
                    "documentid": "id:mynamespace:music::love-id-here-to-stay",
                    "artist": "Diana Krall",
                    "album": "Love Is Here To Stay",
                    "year": 2018,
                    "category_scores": {
                        "cells": [
                            {
                                "address": {
                                    "cat": "pop"
                                },
                                "value": 0.4000000059604645
                            },
                            {
                                "address": {
                                    "cat": "rock"
                                },
                                "value": 0.0
                            },
                            {
                                "address": {
                                    "cat": "jazz"
                                },
                                "value": 0.800000011920929
                            }
                        ]
                    }
                }
            },
            {
                "id": "id:mynamespace:music::hardwired-to-self-destruct",
                "relevance": 0.20000000298023224,
                "source": "music",
                "fields": {
                    "sddocname": "music",
                    "documentid": "id:mynamespace:music::hardwired-to-self-destruct",
                    "artist": "Metallica",
                    "album": "Hardwired...To Self-Destruct",
                    "year": 2016,
                    "category_scores": {
                        "cells": [
                            {
                                "address": {
                                    "cat": "pop"
                                },
                                "value": 0.0
                            },
                            {
                                "address": {
                                    "cat": "rock"
                                },
                                "value": 1.0
                            },
                            {
                                "address": {
                                    "cat": "jazz"
                                },
                                "value": 0.0
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```
