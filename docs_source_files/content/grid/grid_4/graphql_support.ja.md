---
title: "GraphQLクエリのサポート"
weight: 4
---

GraphQLは、APIのクエリ言語であり、既存のデータでこれらのクエリを実行するためのランタイムです。
これにより、ユーザーは必要なものだけを正確に要求することができます。

## 列挙型(Enum)
列挙型は、フィールドの可能な値のセットを表します。

たとえば、 `Node` オブジェクトには `status` というフィールドがあります。
`UP` 、 `DRAINING` 、または `UNAVAILABLE` の可能性があるため、状態は、 列挙型（具体的には、`Status` タイプ）です。

## スカラー
スカラーはプリミティブ値です： `Int` 、 `Float` 、 `String` 、 `Boolean` 、または `ID` 。

GraphQL APIを呼び出すときは、スカラーのみを返すまでネストされたサブフィールドを指定する必要があります。

## スキーマの構造
グリッドスキーマの構造は次のとおりです。

```shell
{
    session(id: "<session-id>") : {
        id,
        capabilities,
        startTime,
        uri,
        nodeId,
        nodeUri,
        sessionDurationMillis
        slot : {
            id,
            stereotype,
            lastStarted
        }
    }
    grid: {
        uri,
        totalSlots,
        nodeCount,
        maxSession,
        sessionCount,
        version,
        sessionQueueSize
    }
    sessionsInfo: {
        sessionQueueRequests,
        sessions: [
            {
                id,
                capabilities,
                startTime,
                uri,
                nodeId,
                nodeUri,
                sessionDurationMillis
                slot : {
                    id,
                    stereotype,
                    lastStarted
                }
            }
        ]
    }
    nodesInfo: {
        nodes : [
            {
                id,
                uri,
                status,
                maxSession,
                slotCount,
                sessions: [
                    {
                        id,
                        capabilities,
                        startTime,
                        uri,
                        nodeId,
                        nodeUri,
                        sessionDurationMillis
                        slot : {
                            id,
                            stereotype,
                            lastStarted
                        }
                    }
                ],
                sessionCount,
                stereotypes,
                version,
                osInfo: {
                    arch,
                    name,
                    version
                }
            }
        ]
    }
}
```
## GraphQLで照会する

GraphQLをクエリする最良の方法は、 `curl` リクエストを使用することです。
GraphQLを使用すると、必要なデータのみをフェッチできます。それ以上でもそれ以下でもありません。

Some of the example GraphQL queries are given below. You can build your own queries as you like.

### グリッド内の `maxSession`と` sessionCount`の数を照会する

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query": "{ grid { maxSession, sessionCount } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

通常、ローカルマシンでは、 `<LINK_TO_GRAPHQL_ENDPOINT>` は `http://localhost:4444/graphql` になります。

### Querying all details for session, node and the Grid :

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ grid { uri, maxSession, sessionCount }, nodesInfo { nodes { id, uri, status, sessions { id, capabilities, startTime, uri, nodeId, nodeUri, sessionDurationMillis, slot { id, stereotype, lastStarted } }, slotCount, sessionCount }} }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query for getting the current session count in the Grid :

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ grid { sessionCount } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query for getting the max session count in the Grid :

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ grid { maxSession } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query for getting all session details for all nodes in the Grid :

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ sessionsInfo { sessions { id, capabilities, startTime, uri, nodeId, nodeId, sessionDurationMillis } } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query to get slot information for all sessions in each Node in the Grid :

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ sessionsInfo { sessions { id, slot { id, stereotype, lastStarted } } } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query to get session information for a given session:

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ session (id: "<session-id>") { id, capabilities, startTime, uri, nodeId, nodeUri, sessionDurationMillis, slot { id, stereotype, lastStarted } } } "}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### グリッド内の各ノードのcapabilityを照会する

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query": "{ nodesInfo { nodes { stereotypes } } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### グリッド内の各ノードのステータスを照会する

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query": "{ nodesInfo { nodes { status } } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### 各ノードとグリッドのURIを照会する

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query": "{ nodesInfo { nodes { uri } } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query for getting the current requests in the New Session Queue:

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ sessionsInfo { sessionQueueRequests } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```

### Query for getting the New Session Queue size :

```shell
curl -X POST -H "Content-Type: application/json" --data '{"query":"{ grid { sessionQueueSize } }"}' -s <LINK_TO_GRAPHQL_ENDPOINT>
```