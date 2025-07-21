---
id: network-mode
title: Network Mode
---

<span title="TanStack Query는 네트워크 연결이 없을 때 쿼리와 뮤테이션이 어떻게 동작해야 하는지 구분하기 위해 세 가지 네트워크 모드를 제공합니다. 이 모드는 각 쿼리/뮤테이션별로 개별 설정하거나, 전역 기본값으로 설정할 수 있습니다.">TanStack Query provides three different network modes to distinguish how [Queries](../queries.md) and [Mutations](../mutations.md) should behave if you have no network connection. This mode can be set for each Query / Mutation individually, or globally via the query / mutation defaults.</span>

<span title="TanStack Query는 주로 데이터 패칭 라이브러리와 함께 사용되므로, 기본 네트워크 모드는 online입니다.">Since TanStack Query is most often used for data fetching in combination with data fetching libraries, the default network mode is [online](#network-mode-online).</span>

## <span title="네트워크 모드: online">Network Mode: online</span>

<span title="이 모드에서는 네트워크 연결이 없으면 쿼리와 뮤테이션이 실행되지 않습니다. 이게 기본 모드입니다. 쿼리의 fetch가 시작되면, 네트워크가 없어서 fetch를 할 수 없는 경우 쿼리는 항상 현재 상태(pending, error, success)에 머무릅니다. 단, fetchStatus가 추가로 노출됩니다. fetchStatus는 다음 중 하나일 수 있습니다:">In this mode, Queries and Mutations will not fire unless you have network connection. This is the default mode. If a fetch is initiated for a query, it will always stay in the `state` (`pending`, `error`, `success`) it is in if the fetch cannot be made because there is no network connection. However, a [fetchStatus](../queries.md#fetchstatus) is exposed additionally. This can be either:</span>

- <span title="queryFn이 실제로 실행 중(요청이 진행 중)">`fetching`: The `queryFn` is really executing - a request is in-flight.</span>
- <span title="쿼리가 실행되지 않고, 네트워크 연결을 기다리며 일시정지됨">`paused`: The query is not executing - it is `paused` until you have connection again</span>
- <span title="쿼리가 fetch 중도 아니고, 일시정지 상태도 아님">`idle`: The query is not fetching and not paused</span>

<span title="isFetching과 isPaused 플래그는 이 상태에서 파생되어 편의상 노출됩니다.">The flags `isFetching` and `isPaused` are derived from this state and exposed for convenience.</span>

> <span title="로딩 스피너를 보여주기 위해 pending 상태만 체크하는 것으로는 충분하지 않을 수 있습니다. 쿼리가 처음 마운트될 때 네트워크가 없으면 state: 'pending'이면서 fetchStatus: 'paused'일 수 있습니다.">Keep in mind that it might not be enough to check for `pending` state to show a loading spinner. Queries can be in `state: 'pending'`, but `fetchStatus: 'paused'` if they are mounting for the first time, and you have no network connection.</span>

<span title="쿼리가 온라인 상태에서 실행되다가, fetch 중에 오프라인이 되면 TanStack Query는 재시도 메커니즘도 일시정지합니다. 일시정지된 쿼리는 네트워크가 복구되면 다시 실행됩니다. 이는 refetchOnReconnect와는 별개(이 모드에서 기본값 true)로, refetch가 아니라 continue이기 때문입니다. 쿼리가 그 사이에 취소되었다면 계속되지 않습니다.">If a query runs because you are online, but you go offline while the fetch is still happening, TanStack Query will also pause the retry mechanism. Paused queries will then continue to run once you re-gain network connection. This is independent of `refetchOnReconnect` (which also defaults to `true` in this mode), because it is not a `refetch`, but rather a `continue`. If the query has been [cancelled](../query-cancellation.md) in the meantime, it will not continue.</span>

## <span title="네트워크 모드: always">Network Mode: always</span>

<span title="이 모드에서는 TanStack Query가 항상 fetch를 시도하며, 온라인/오프라인 상태를 무시합니다. 쿼리가 네트워크 연결 없이 동작해야 하는 환경(예: AsyncStorage만 읽거나, queryFn에서 Promise.resolve(5)만 반환하는 경우)에 적합합니다.">In this mode, TanStack Query will always fetch and ignore the online / offline state. This is likely the mode you want to choose if you use TanStack Query in an environment where you don't need an active network connection for your Queries to work - e.g. if you just read from `AsyncStorage`, or if you just want to return `Promise.resolve(5)` from your `queryFn`.</span>

- <span title="네트워크 연결이 없어도 쿼리가 일시정지되지 않습니다.">Queries will never be `paused` because you have no network connection.</span>
- <span title="재시도도 일시정지되지 않고, 실패하면 쿼리는 error 상태로 갑니다.">Retries will also not pause - your Query will go to `error` state if it fails.</span>
- <span title="이 모드에서는 refetchOnReconnect의 기본값이 false입니다. 네트워크가 복구되어도 stale 쿼리를 refetch해야 한다는 보장이 없기 때문입니다. 원한다면 켤 수 있습니다.">`refetchOnReconnect` defaults to `false` in this mode, because reconnecting to the network is not a good indicator anymore that stale queries should be refetched. You can still turn it on if you want.</span>

## <span title="네트워크 모드: offlineFirst">Network Mode: offlineFirst</span>

<span title="이 모드는 앞의 두 옵션의 중간 지점으로, TanStack Query가 queryFn을 한 번 실행한 뒤에는 재시도를 일시정지합니다. 서비스워커가 캐싱을 가로채는 오프라인 퍼스트 PWA나, HTTP 캐싱을 사용하는 경우에 유용합니다.">This mode is the middle ground between the first two options, where TanStack Query will run the `queryFn` once, but then pause retries. This is very handy if you have a serviceWorker that intercepts a request for caching like in an [offline-first PWA](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Offline_Service_workers), or if you use HTTP caching via the [Cache-Control header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#the_cache-control_header).</span>

<span title="이런 상황에서는 첫 fetch는 오프라인 저장소/캐시에서 성공할 수 있습니다. 하지만 캐시 미스가 나면 네트워크 요청이 나가고 실패할 수 있는데, 이 경우 이 모드는 online 쿼리처럼 동작하며 재시도를 일시정지합니다.">In those situations, the first fetch might succeed because it comes from an offline storage / cache. However, if there is a cache miss, the network request will go out and fail, in which case this mode behaves like an `online` query - pausing retries.</span>

## <span title="Devtools">Devtools</span>

<span title="TanStack Query Devtools는 네트워크 연결이 없어서 fetch를 못하는 쿼리를 paused 상태로 보여줍니다. 오프라인 동작을 모킹하는 토글 버튼도 있습니다. 이 버튼은 실제 네트워크 연결을 끊지는 않고, OnlineManager를 오프라인 상태로 만듭니다.">The [TanStack Query Devtools](../../devtools.md) will show Queries in a `paused` state if they would be fetching, but there is no network connection. There is also a toggle button to _Mock offline behavior_. Please note that this button will _not_ actually mess with your network connection (you can do that in the browser devtools), but it will set the [OnlineManager](../../../../reference/onlineManager.md) in an offline state.</span>

## <span title="시그니처">Signature</span>

- <span title="networkMode: 'online' | 'always' | 'offlineFirst' (옵션, 기본값은 'online')">`networkMode: 'online' | 'always' | 'offlineFirst'`
  - optional
  - defaults to `'online'`</span>

## 요약

- TanStack Query는 네트워크 연결 상태에 따라 쿼리/뮤테이션의 동작을 제어할 수 있는 세 가지 네트워크 모드(online, always, offlineFirst)를 제공합니다.
- online 모드는 네트워크가 없으면 쿼리가 실행되지 않고, fetchStatus로 paused/fetching/idle 상태를 구분합니다.
- always 모드는 네트워크 연결 여부와 상관없이 항상 fetch를 시도하며, 오프라인 환경이나 캐시 기반 쿼리에 적합합니다.
- offlineFirst 모드는 첫 fetch는 시도하지만, 이후 재시도는 네트워크가 복구될 때까지 일시정지합니다. 오프라인 PWA, HTTP 캐시 등과 잘 어울립니다.
- Devtools에서 네트워크 상태에 따른 쿼리의 paused 상태를 시각적으로 확인할 수 있으며, 시그니처 옵션으로 손쉽게 모드를 설정할 수 있습니다.
