---
id: important-defaults
title: Important Defaults
---

<span title="TanStack Query는 기본적으로 공격적이지만 합리적인 기본값으로 설정되어 있습니다.">Out of the box, TanStack Query is configured with **aggressive but sane** defaults.</span> <span title="이러한 기본값은 새로운 사용자를 당황하게 하거나, 모를 경우 학습/디버깅을 어렵게 만들 수 있습니다.">**Sometimes these defaults can catch new users off guard or make learning/debugging difficult if they are unknown by the user.**</span> <span title="TanStack Query를 계속 배우고 사용할 때 이 점을 꼭 기억하세요.">Keep them in mind as you continue to learn and use TanStack Query:</span>

- <span title="useQuery 또는 useInfiniteQuery를 통해 생성된 쿼리 인스턴스는 기본적으로 캐시된 데이터를 오래된(stale) 것으로 간주합니다.">Query instances via `useQuery` or `useInfiniteQuery` by default **consider cached data as stale**.</span>

> <span title="이 동작을 변경하려면, staleTime 옵션을 사용해 전역 또는 쿼리별로 쿼리를 설정할 수 있습니다. 더 긴 staleTime을 지정하면 쿼리가 데이터를 자주 다시 가져오지 않습니다.">To change this behavior, you can configure your queries both globally and per-query using the `staleTime` option. Specifying a longer `staleTime` means queries will not refetch their data as often</span>

- <span title="staleTime이 설정된 쿼리는 해당 시간이 경과할 때까지 신선(fresh)하다고 간주됩니다.">A Query that has a `staleTime` set is considered **fresh** until that `staleTime` has elapsed.</span>
  - <span title="예를 들어 staleTime을 2 * 60 * 1000으로 설정하면, 2분 동안 또는 쿼리가 수동으로 무효화될 때까지 데이터가 캐시에서 읽히고, 어떤 재요청도 발생하지 않습니다.">set `staleTime` to e.g. `2 * 60 * 1000` to make sure data is read from the cache, without triggering any kinds of refetches, for 2 minutes, or until the Query is [invalidated manually](../query-invalidation.md).</span>
  - <span title="staleTime을 Infinity로 설정하면 쿼리가 수동으로 무효화될 때까지 재요청이 발생하지 않습니다.">set `staleTime` to `Infinity` to never trigger a refetch until the Query is [invalidated manually](../query-invalidation.md).</span>
  - <span title="staleTime을 'static'으로 설정하면 쿼리가 수동으로 무효화되어도 절대 재요청이 발생하지 않습니다.">set `staleTime` to `'static'` to **never** trigger a refetch, even if the Query is [invalidated manually](../query-invalidation.md).</span>

- <span title="오래된 쿼리는 다음과 같은 경우 백그라운드에서 자동으로 다시 요청됩니다:">Stale queries are refetched automatically in the background when:</span>
  - <span title="쿼리의 새로운 인스턴스가 마운트될 때">New instances of the query mount</span>
  - <span title="윈도우가 다시 포커스될 때">The window is refocused</span>
  - <span title="네트워크가 다시 연결될 때">The network is reconnected</span>

> <span title="과도한 재요청을 피하려면 staleTime을 설정하는 것이 권장되지만, refetchOnMount, refetchOnWindowFocus, refetchOnReconnect와 같은 옵션을 통해 재요청 시점을 커스터마이즈할 수도 있습니다.">Setting `staleTime` is the recommended way to avoid excessive refetches, but you can also customize the points in time for refetches by setting options like `refetchOnMount`, `refetchOnWindowFocus` and `refetchOnReconnect`.</span>

- <span title="쿼리는 선택적으로 refetchInterval을 설정하여 주기적으로 재요청을 트리거할 수 있으며, 이는 staleTime 설정과는 별개입니다.">Queries can optionally be configured with a `refetchInterval` to trigger refetches periodically, which is independent of the `staleTime` setting.</span>

- <span title="useQuery, useInfiniteQuery 또는 쿼리 옵저버의 활성 인스턴스가 더 이상 없는 쿼리 결과는 '비활성(inactive)'으로 표시되며, 나중에 다시 사용될 경우를 대비해 캐시에 남아 있습니다.">Query results that have no more active instances of `useQuery`, `useInfiniteQuery` or query observers are labeled as "inactive" and remain in the cache in case they are used again at a later time.</span>
- <span title="기본적으로 '비활성' 쿼리는 5분 후에 가비지 컬렉션됩니다.">By default, "inactive" queries are garbage collected after **5 minutes**.</span>

> <span title="이 동작을 변경하려면 쿼리의 기본 gcTime을 1000 * 60 * 5 밀리초가 아닌 값으로 설정할 수 있습니다.">To change this, you can alter the default `gcTime` for queries to something other than `1000 * 60 * 5` milliseconds.</span>

- <span title="실패한 쿼리는 UI에 에러가 표시되기 전에 지수 백오프(exponential backoff) 딜레이와 함께 3번 조용히 재시도됩니다.">Queries that fail are **silently retried 3 times, with exponential backoff delay** before capturing and displaying an error to the UI.</span>

> <span title="이 동작을 변경하려면 쿼리의 기본 retry와 retryDelay 옵션을 3과 기본 지수 백오프 함수가 아닌 값으로 설정할 수 있습니다.">To change this, you can alter the default `retry` and `retryDelay` options for queries to something other than `3` and the default exponential backoff function.</span>

- <span title="쿼리 결과는 기본적으로 데이터가 실제로 변경되었는지 감지하기 위해 구조적으로 공유(structural sharing)되며, 변경이 없다면 데이터 참조가 그대로 유지되어 useMemo, useCallback 등에서 값 안정성에 도움이 됩니다. 이 개념이 낯설다면 걱정하지 마세요! 99.9%의 경우 이 기능을 비활성화할 필요가 없으며, 성능 향상에 도움이 됩니다.">Query results by default are **structurally shared to detect if data has actually changed** and if not, **the data reference remains unchanged** to better help with value stabilization with regards to useMemo and useCallback. If this concept sounds foreign, then don't worry about it! 99.9% of the time you will not need to disable this and it makes your app more performant at zero cost to you.</span>

> <span title="구조적 공유는 JSON 호환 값에만 동작하며, 그 외의 값 타입은 항상 변경된 것으로 간주됩니다. 예를 들어, 응답이 너무 커서 성능 문제가 발생한다면 config.structuralSharing 플래그로 이 기능을 비활성화할 수 있습니다. 쿼리 응답에 JSON 비호환 값이 포함되어 있고, 데이터 변경 여부를 감지하고 싶다면, config.structuralSharing에 직접 커스텀 함수를 제공해 이전/새 응답에서 값을 계산하고 참조를 유지할 수 있습니다.">Structural sharing only works with JSON-compatible values, any other value types will always be considered as changed. If you are seeing performance issues because of large responses for example, you can disable this feature with the `config.structuralSharing` flag. If you are dealing with non-JSON compatible values in your query responses and still want to detect if data has changed or not, you can provide your own custom function as `config.structuralSharing` to compute a value from the old and new responses, retaining references as required.</span>

[//]: # 'Materials'

## <span title="추가 자료">Further Reading</span>

<span title="기본값에 대한 추가 설명은 아래 커뮤니티 리소스의 글을 참고하세요:">Have a look at the following articles from our Community Resources for further explanations of the defaults:</span>

- <span title="Practical React Query">[Practical React Query](../../community/tkdodos-blog.md#1-practical-react-query)</span>
- <span title="React Query as a State Manager">[React Query as a State Manager](../../community/tkdodos-blog.md#10-react-query-as-a-state-manager)</span>

[//]: # 'Materials'

## 요약

- TanStack Query는 기본적으로 데이터를 공격적으로 stale로 간주하고, refetch를 자주 트리거합니다.
- staleTime, gcTime, retry, structuralSharing 등 주요 옵션을 통해 쿼리의 신선도, 캐싱, 에러 처리, 성능을 세밀하게 제어할 수 있습니다.
- 쿼리는 네트워크/포커스/마운트 등 다양한 시점에 자동으로 refetch되며, 필요에 따라 refetchOnMount, refetchOnWindowFocus, refetchOnReconnect 등으로 커스터마이즈할 수 있습니다.
- 쿼리의 비활성 상태, 가비지 컬렉션, 구조적 공유 등은 메모리 관리와 성능 최적화에 중요한 역할을 합니다.
- 대부분의 기본값은 실전에서 안전하고 효율적이지만, 앱의 특성에 맞게 옵션을 조정하는 것이 중요합니다.
