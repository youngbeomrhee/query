# tanstack query staleTime deep dive

## TanStack Query의 staleTime 구현 분석

TanStack Query의 staleTime 관련 구현을 코드베이스에서 살펴보겠습니다.

### 1. 타입 정의

```12:15:packages/query-core/src/types.ts
export type StaleTime = number | 'static'

export type StaleTimeFunction<
  TQueryFnData = unknown,
  TError = DefaultError,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey,
> =
  | StaleTime
  | ((query: Query<TQueryFnData, TError, TData, TQueryKey>) => StaleTime)
```

staleTime은 다음 세 가지 형태로 사용됩니다:
- **number**: 밀리초 단위의 stale 시간
- **'static'**: 수동으로 무효화되지 않는 한 절대 stale하지 않음
- **함수**: 쿼리 상태를 기반으로 동적으로 staleTime을 결정

### 2. 핵심 유틸리티 함수

```95:112:packages/query-core/src/utils.ts
export function timeUntilStale(updatedAt: number, staleTime?: number): number {
  return Math.max(updatedAt + (staleTime || 0) - Date.now(), 0)
}

export function resolveStaleTime<
  TQueryFnData = unknown,
  TError = DefaultError,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey,
>(
  staleTime:
    | undefined
    | StaleTimeFunction<TQueryFnData, TError, TData, TQueryKey>,
  query: Query<TQueryFnData, TError, TData, TQueryKey>,
): StaleTime | undefined {
  return typeof staleTime === 'function' ? staleTime(query) : staleTime
}
```

- **timeUntilStale()**: 데이터 업데이트 시간 + staleTime - 현재 시간을 계산하여 stale해지기까지 남은 시간을 반환
- **resolveStaleTime()**: 함수형 staleTime을 실제 값으로 변환

### 3. Query 클래스의 staleTime 관련 메서드

```296:312:packages/query-core/src/query.ts
isStaleByTime(staleTime: StaleTime = 0): boolean {
  // no data is always stale
  if (this.state.data === undefined) {
    return true
  }
  // static is never stale
  if (staleTime === 'static') {
    return false
  }
  // if the query is invalidated, it is stale
  if (this.state.isInvalidated) {
    return true
  }

  return !timeUntilStale(this.state.dataUpdatedAt, staleTime)
}
```

핵심 로직:
1. 데이터가 없으면 항상 stale
2. staleTime이 'static'이면 절대 stale하지 않음
3. 쿼리가 invalidated되면 stale
4. 그 외의 경우 시간 기반으로 stale 여부 결정

### 4. QueryObserver의 staleTime 관리

```344:367:packages/query-core/src/queryObserver.ts
#updateStaleTimeout(): void {
  this.#clearStaleTimeout()
  const staleTime = resolveStaleTime(
    this.options.staleTime,
    this.#currentQuery,
  )

  if (isServer || this.#currentResult.isStale || !isValidTimeout(staleTime)) {
    return
  }

  const time = timeUntilStale(this.#currentResult.dataUpdatedAt, staleTime)

  // The timeout is sometimes triggered 1 ms before the stale time expiration.
  // To mitigate this issue we always add 1 ms to the timeout.
  const timeout = time + 1

  this.#staleTimeoutId = setTimeout(() => {
    if (!this.#currentResult.isStale) {
      this.updateResult()
    }
  }, timeout)
}
```

Observer는 setTimeout을 사용하여 데이터가 stale해지는 시점을 정확히 추적합니다. 1ms를 추가하여 타이밍 이슈를 방지합니다.

### 5. staleTime 변경 감지

```203:211:packages/query-core/src/queryObserver.ts
if (
  mounted &&
  (this.#currentQuery !== prevQuery ||
    resolveEnabled(this.options.enabled, this.#currentQuery) !==
      resolveEnabled(prevOptions.enabled, this.#currentQuery) ||
    resolveStaleTime(this.options.staleTime, this.#currentQuery) !==
      resolveStaleTime(prevOptions.staleTime, this.#currentQuery))
) {
  this.#updateStaleTimeout()
}
```

staleTime이 변경되면 기존 timeout을 해제하고 새로운 timeout을 설정합니다.

### 6. QueryClient의 staleTime 활용

```364:370:packages/query-core/src/queryClient.ts
return query.isStaleByTime(
  resolveStaleTime(defaultedOptions.staleTime, query),
)
  ? query.fetch(defaultedOptions)
  : Promise.resolve(query.state.data as TData)
```

fetchQuery에서 staleTime을 확인하여 캐시된 데이터를 반환할지 새로 fetch할지 결정합니다.

### 7. 특별한 케이스: 'static' staleTime

테스트 코드에서 확인할 수 있듯이:

```662:688:packages/query-core/src/__tests__/queryClient.test.tsx
test('should read from cache with static staleTime even if invalidated', async () => {
  const key = queryKey()

  const fetchFn = vi.fn(() => Promise.resolve({ data: 'data' }))
  const first = await queryClient.fetchQuery({
    queryKey: key,
    queryFn: fetchFn,
    staleTime: 'static',
  })

  expect(first.data).toBe('data')
  expect(fetchFn).toHaveBeenCalledTimes(1)

  await queryClient.invalidateQueries({
    queryKey: key,
    refetchType: 'none',
  })

  const second = await queryClient.fetchQuery({
    queryKey: key,
    queryFn: fetchFn,
    staleTime: 'static',
  })

  expect(fetchFn).toHaveBeenCalledTimes(1)
  expect(second).toBe(first)
})
```

'static' staleTime을 사용하면 invalidate되어도 refetch가 발생하지 않습니다.

### 8. 함수형 staleTime 사용

```1161:1189:packages/query-core/src/__tests__/queryObserver.test.tsx
test('should allow staleTime as a function', async () => {
  const key = queryKey()
  const observer = new QueryObserver(queryClient, {
    queryKey: key,
    queryFn: async () => {
      await sleep(5)
      return {
        data: 'data',
        staleTime: 20,
      }
    },
    staleTime: (query) => query.state.data?.staleTime ?? 0,
  })
  // ... 테스트 코드
})
```

함수형 staleTime을 사용하면 쿼리 상태에 따라 동적으로 staleTime을 조정할 수 있습니다.

## 핵심 동작 원리

1. **데이터 패칭 시**: `isStaleByTime`으로 데이터 freshness 확인
2. **타이머 설정**: `#updateStaleTimeout`으로 정확한 stale 시점 추적
3. **옵션 변경 감지**: staleTime 변경 시 타이머 재설정
4. **특별 케이스 처리**: 'static' 모드에서는 수동 무효화만 허용

이 구현을 통해 TanStack Query는 효율적인 캐시 관리와 자동 refetch 기능을 제공하며, 개발자가 원하는 수준의 데이터 freshness를 유지할 수 있도록 합니다.
