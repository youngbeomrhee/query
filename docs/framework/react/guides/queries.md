---
id: queries
title: Queries
---

## <span title="쿼리의 기본">Query Basics</span>

<span title="쿼리는 비동기 데이터 소스에 선언적으로 의존하며, 고유 키에 연결됩니다. 쿼리는 모든 Promise 기반 메서드(GET, POST 등)로 서버에서 데이터를 가져오는 데 사용할 수 있습니다. 만약 서버의 데이터를 수정하는 메서드라면, Mutations를 사용하는 것이 좋습니다.">A query is a declarative dependency on an asynchronous source of data that is tied to a **unique key**. A query can be used with any Promise based method (including GET and POST methods) to fetch data from a server. If your method modifies data on the server, we recommend using [Mutations](../mutations.md) instead.</span>

<span title="컴포넌트나 커스텀 훅에서 쿼리를 구독하려면, 최소한 다음과 같이 useQuery 훅을 호출하세요.">To subscribe to a query in your components or custom hooks, call the `useQuery` hook with at least:</span>

- <span title="쿼리의 고유 키">A **unique key for the query**</span>
- <span title="Promise를 반환하는 함수(데이터를 resolve하거나 에러를 throw)">A function that returns a promise that:</span>
  - <span title="데이터를 resolve">Resolves the data, or</span>
  - <span title="에러를 throw">Throws an error</span>

[//]: # 'Example'

```tsx
import { useQuery } from '@tanstack/react-query'

function App() {
  const info = useQuery({ queryKey: ['todos'], queryFn: fetchTodoList })
}
```

[//]: # 'Example'

<span title="제공한 고유 키는 내부적으로 리패칭, 캐싱, 쿼리 공유 등에 사용됩니다.">The **unique key** you provide is used internally for refetching, caching, and sharing your queries throughout your application.</span>

<span title="useQuery가 반환하는 쿼리 결과에는 템플릿 작성이나 데이터 활용에 필요한 모든 정보가 담겨 있습니다.">The query result returned by `useQuery` contains all of the information about the query that you'll need for templating and any other usage of the data:</span>

[//]: # 'Example2'

```tsx
const result = useQuery({ queryKey: ['todos'], queryFn: fetchTodoList })
```

[//]: # 'Example2'

<span title="result 객체에는 생산성을 위해 반드시 알아야 할 중요한 상태들이 들어 있습니다. 쿼리는 언제든 다음 상태 중 하나만 가질 수 있습니다.">The `result` object contains a few very important states you'll need to be aware of to be productive. A query can only be in one of the following states at any given moment:</span>

- <span title="아직 데이터가 없는 상태">`isPending` or `status === 'pending'` - The query has no data yet</span>
- <span title="쿼리에서 에러가 발생한 상태">`isError` or `status === 'error'` - The query encountered an error</span>
- <span title="쿼리가 성공적으로 완료되어 데이터가 있는 상태">`isSuccess` or `status === 'success'` - The query was successful and data is available</span>

<span title="이 주요 상태 외에도, 쿼리의 상태에 따라 더 많은 정보가 제공됩니다.">Beyond those primary states, more information is available depending on the state of the query:</span>

- <span title="isError 상태일 때 error 프로퍼티로 에러에 접근 가능">`error` - If the query is in an `isError` state, the error is available via the `error` property.</span>
- <span title="isSuccess 상태일 때 data 프로퍼티로 데이터에 접근 가능">`data` - If the query is in an `isSuccess` state, the data is available via the `data` property.</span>
- <span title="어떤 상태에서든 쿼리가 fetch 중이면 isFetching이 true">`isFetching` - In any state, if the query is fetching at any time (including background refetching) `isFetching` will be `true`.</span>

<span title="대부분의 쿼리에서는 isPending, isError, 그 다음 성공 상태 순으로 체크하면 충분합니다.">For **most** queries, it's usually sufficient to check for the `isPending` state, then the `isError` state, then finally, assume that the data is available and render the successful state:</span>

[//]: # 'Example3'

```tsx
function Todos() {
  const { isPending, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  })

  if (isPending) {
    return <span>Loading...</span>
  }

  if (isError) {
    return <span>Error: {error.message}</span>
  }

  // We can assume by this point that `isSuccess === true`
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

[//]: # 'Example3'

<span title="불리언 대신 status 상태를 사용할 수도 있습니다.">If booleans aren't your thing, you can always use the `status` state as well:</span>

[//]: # 'Example4'

```tsx
function Todos() {
  const { status, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  })

  if (status === 'pending') {
    return <span>Loading...</span>
  }

  if (status === 'error') {
    return <span>Error: {error.message}</span>
  }

  // also status === 'success', but "else" logic works, too
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

[//]: # 'Example4'

<span title="pending과 error를 먼저 체크하면 TypeScript가 data 타입을 자동으로 좁혀줍니다.">TypeScript will also narrow the type of `data` correctly if you've checked for `pending` and `error` before accessing it.</span>

### <span title="FetchStatus">FetchStatus</span>

<span title="status 필드 외에도 fetchStatus 프로퍼티가 추가로 제공됩니다.">In addition to the `status` field, you will also get an additional `fetchStatus` property with the following options:</span>

- <span title="현재 쿼리가 fetch 중">`fetchStatus === 'fetching'` - The query is currently fetching.</span>
- <span title="쿼리가 fetch를 원하지만 일시정지됨">`fetchStatus === 'paused'` - The query wanted to fetch, but it is paused. Read more about this in the [Network Mode](../network-mode.md) guide.</span>
- <span title="쿼리가 아무것도 하지 않는 상태">`fetchStatus === 'idle'` - The query is not doing anything at the moment.</span>

### <span title="왜 두 가지 상태가 있나요?">Why two different states?</span>

<span title="백그라운드 refetch와 stale-while-revalidate 로직 때문에 status와 fetchStatus의 모든 조합이 가능합니다. 예시:">Background refetches and stale-while-revalidate logic make all combinations for `status` and `fetchStatus` possible. For example:</span>

- <span title="성공 상태의 쿼리는 보통 idle fetchStatus지만, 백그라운드 refetch 중이면 fetching일 수 있습니다.">a query in `success` status will usually be in `idle` fetchStatus, but it could also be in `fetching` if a background refetch is happening.</span>
- <span title="마운트되고 데이터가 없는 쿼리는 보통 pending/pending이지만, 네트워크가 없으면 paused일 수 있습니다.">a query that mounts and has no data will usually be in `pending` status and `fetching` fetchStatus, but it could also be `paused` if there is no network connection.</span>

<span title="즉, 쿼리가 실제로 데이터를 fetch하지 않아도 pending 상태일 수 있습니다. 정리하면:">So keep in mind that a query can be in `pending` state without actually fetching data. As a rule of thumb:</span>

- <span title="status는 데이터에 대한 정보(데이터가 있는가?)">The `status` gives information about the `data`: Do we have any or not?</span>
- <span title="fetchStatus는 queryFn에 대한 정보(실행 중인가?)">The `fetchStatus` gives information about the `queryFn`: Is it running or not?</span>

## <span title="완전한 상태 레퍼런스">Complete State Reference</span>

<span title="useQuery가 반환하는 모든 상태값들과 그 의미를 체계적으로 정리했습니다.">Here's a comprehensive reference of all state values returned by `useQuery` and their meanings.</span>

### <span title="기본 상태 (Primary States)">Primary States</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `status` | `'pending' \| 'error' \| 'success'` | <span title="쿼리의 전반적인 상태를 나타냅니다. 데이터의 유무를 판단하는 주요 지표입니다.">Overall query state. Primary indicator of data availability.</span> |
| `fetchStatus` | `'fetching' \| 'paused' \| 'idle'` | <span title="queryFn의 실행 상태를 나타냅니다. 네트워크 요청의 진행 상황을 추적합니다.">Execution state of queryFn. Tracks network request progress.</span> |

### <span title="불리언 상태 지표 (Boolean State Indicators)">Boolean State Indicators</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `isPending` | `boolean` | <span title="status === 'pending'와 동일. 아직 데이터가 없는 상태입니다.">Equivalent to `status === 'pending'`. No data available yet.</span> |
| `isError` | `boolean` | <span title="status === 'error'와 동일. 쿼리 실행 중 오류가 발생한 상태입니다.">Equivalent to `status === 'error'`. Query encountered an error.</span> |
| `isSuccess` | `boolean` | <span title="status === 'success'와 동일. 쿼리가 성공적으로 완료되어 데이터가 있는 상태입니다.">Equivalent to `status === 'success'`. Query completed successfully with data.</span> |
| `isFetching` | `boolean` | <span title="fetchStatus === 'fetching'와 동일. 현재 쿼리가 실행 중인 상태입니다 (초기 로딩 또는 백그라운드 리페치 포함).">Equivalent to `fetchStatus === 'fetching'`. Query is currently executing (initial or background).</span> |
| `isLoading` | `boolean` | <span title="첫 번째 패치가 진행 중인 상태입니다. isFetching && isPending과 동일합니다.">First fetch for a query is in-flight. Same as `isFetching && isPending`.</span> |
| `isRefetching` | `boolean` | <span title="데이터가 있는 상태에서 백그라운드로 새 데이터를 가져오는 중입니다. isFetching && !isPending과 동일합니다.">Background refetch is in-flight. Same as `isFetching && !isPending`.</span> |
| `isStale` | `boolean` | <span title="데이터가 staleTime을 초과하여 오래된 것으로 간주되는 상태입니다.">Data is considered stale (older than `staleTime`).</span> |
| `isFetched` | `boolean` | <span title="쿼리가 한 번이라도 패치된 상태입니다.">Query has been fetched at least once.</span> |
| `isFetchedAfterMount` | `boolean` | <span title="컴포넌트 마운트 후 쿼리가 패치된 상태입니다. 이전 캐시 데이터를 숨기는 용도로 사용할 수 있습니다.">Query has been fetched after component mount. Can be used to not show cached data.</span> |
| `isPaused` | `boolean` | <span title="fetchStatus === 'paused'와 동일. 쿼리가 패치를 원하지만 일시정지된 상태입니다.">Equivalent to `fetchStatus === 'paused'`. Query wanted to fetch but is paused.</span> |
| `isEnabled` | `boolean` | <span title="이 옵저버가 활성화되어 있는지 여부입니다.">Whether this observer is enabled.</span> |
| `isPlaceholderData` | `boolean` | <span title="현재 표시되는 데이터가 placeholderData인 상태입니다.">Currently displayed data is `placeholderData`.</span> |
| `isLoadingError` | `boolean` | <span title="초기 로딩 중 에러가 발생한 상태입니다 (데이터가 없는 상태에서의 에러).">Error occurred during initial loading (no data available).</span> |
| `isRefetchError` | `boolean` | <span title="백그라운드 리페치 중 에러가 발생한 상태입니다 (기존 데이터는 유지됨).">Error occurred during background refetch (existing data remains).</span> |
| `isInitialLoading` | `boolean` | <span title="더 이상 사용되지 않음. isLoading을 사용하세요.">**Deprecated:** Use `isLoading` instead. Will be removed in next major version.</span> |

### <span title="데이터 및 에러 정보 (Data and Error Information)">Data and Error Information</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `data` | `TData \| undefined` | <span title="쿼리에서 성공적으로 가져온 데이터입니다. isSuccess가 true일 때만 사용 가능합니다.">Successfully fetched data. Available when `isSuccess` is true.</span> |
| `error` | `TError \| null` | <span title="쿼리 실행 중 발생한 에러 객체입니다. isError가 true일 때 사용 가능합니다.">Error object from failed query. Available when `isError` is true.</span> |
| `dataUpdatedAt` | `number` | <span title="데이터가 마지막으로 성공적으로 업데이트된 시간의 타임스탬프입니다.">Timestamp of when data was last successfully updated.</span> |
| `errorUpdatedAt` | `number` | <span title="에러가 마지막으로 업데이트된 시간의 타임스탬프입니다.">Timestamp of when error was last updated.</span> |
| `refetch` | `Function` | <span title="쿼리를 수동으로 다시 실행하는 함수입니다.">Function to manually refetch the query.</span> |
| `promise` | `Promise<TData>` | <span title="쿼리 데이터로 해결되는 안정적인 Promise입니다. experimental_prefetchInRender 플래그가 필요합니다.">Stable promise resolved with query data. Requires `experimental_prefetchInRender` flag.</span> |

### <span title="재시도 및 실패 정보 (Retry and Failure Information)">Retry and Failure Information</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `failureCount` | `number` | <span title="현재 쿼리 실행에서 발생한 연속된 실패 횟수입니다.">Number of consecutive failures for current query execution.</span> |
| `failureReason` | `TError \| null` | <span title="가장 최근 실패의 원인이 된 에러입니다. 재시도가 더 있을 경우에 유용합니다.">Error that caused the most recent failure. Useful during retry attempts.</span> |
| `errorUpdateCount` | `number` | <span title="이 쿼리에서 에러가 업데이트된 총 횟수입니다.">Total number of times error has been updated for this query.</span> |

### <span title="무한 쿼리 전용 상태 (Infinite Query Specific States)">Infinite Query Specific States</span>

<span title="useInfiniteQuery 사용 시에만 추가로 제공되는 상태들입니다.">Additional states available only when using `useInfiniteQuery`.</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `hasNextPage` | `boolean` | <span title="다음 페이지가 있는지 여부입니다. getNextPageParam의 반환값에 따라 결정됩니다.">Whether next page is available. Determined by `getNextPageParam` return value.</span> |
| `hasPreviousPage` | `boolean` | <span title="이전 페이지가 있는지 여부입니다. getPreviousPageParam의 반환값에 따라 결정됩니다.">Whether previous page is available. Determined by `getPreviousPageParam` return value.</span> |
| `isFetchingNextPage` | `boolean` | <span title="다음 페이지를 가져오는 중인지 여부입니다.">Whether next page is currently being fetched.</span> |
| `isFetchingPreviousPage` | `boolean` | <span title="이전 페이지를 가져오는 중인지 여부입니다.">Whether previous page is currently being fetched.</span> |
| `isFetchNextPageError` | `boolean` | <span title="다음 페이지 패치 중 에러가 발생했는지 여부입니다.">Whether error occurred while fetching next page.</span> |
| `isFetchPreviousPageError` | `boolean` | <span title="이전 페이지 패치 중 에러가 발생했는지 여부입니다.">Whether error occurred while fetching previous page.</span> |
| `fetchNextPage` | `Function` | <span title="다음 페이지를 가져오는 함수입니다.">Function to fetch the next page of results.</span> |
| `fetchPreviousPage` | `Function` | <span title="이전 페이지를 가져오는 함수입니다.">Function to fetch the previous page of results.</span> |

### <span title="상태 조합 패턴 (Common State Combination Patterns)">Common State Combination Patterns</span>

| <span title="시나리오">Scenario</span> | <span title="상태 조합">State Combination</span> | <span title="의미">Meaning</span> |
|----------|------------------|---------|
| <span title="초기 로딩">Initial Loading</span> | `isPending: true, isLoading: true, isFetching: true` | <span title="첫 번째 데이터 로딩 중">Loading data for the first time</span> |
| <span title="백그라운드 리페치">Background Refetch</span> | `isSuccess: true, isRefetching: true, isFetching: true` | <span title="기존 데이터를 표시하면서 새 데이터 가져오는 중">Showing existing data while fetching fresh data</span> |
| <span title="네트워크 연결 대기">Network Paused</span> | `isPending: true, isPaused: true, fetchStatus: 'paused'` | <span title="네트워크 연결을 기다리는 중">Waiting for network connection</span> |
| <span title="오래된 데이터 표시">Showing Stale Data</span> | `isSuccess: true, isStale: true, isFetched: true` | <span title="데이터가 있지만 staleTime을 초과함">Data available but considered stale</span> |
| <span title="재시도 중">Retrying</span> | `isError: true, isFetching: true, failureCount > 0` | <span title="에러 발생 후 재시도 중">Retrying after an error occurred</span> |
| <span title="플레이스홀더 표시">Showing Placeholder</span> | `isPending: true, isPlaceholderData: true` | <span title="실제 데이터 로딩 중 임시 데이터 표시">Showing temporary data while loading real data</span> |
| <span title="성공 후 데이터 표시">Successfully Loaded</span> | `isSuccess: true, isFetched: true, isFetchedAfterMount: true` | <span title="데이터를 성공적으로 로드하여 표시 중">Successfully loaded and displaying data</span> |

<span title="이 표를 참조하여 애플리케이션의 요구사항에 맞는 적절한 상태 조합을 선택할 수 있습니다.">Use this reference to select appropriate state combinations for your application's requirements.</span>

## <span title="실제 사용 예시들">Practical Usage Examples</span>

<span title="TanStack Query의 다양한 상태 조합들이 실제로 어떤 문제를 해결하는지 구체적인 예시를 통해 살펴보겠습니다.">Let's explore how TanStack Query's various state combinations solve real-world problems through concrete examples.</span>

### <span title="1. 초기 로딩과 백그라운드 리페치 구분">1. Distinguishing Initial Loading from Background Refetch</span>

**<span title="문제 상황">Problem:</span>** <span title="사용자가 이미 데이터를 본 상태에서 백그라운드로 새 데이터를 가져올 때, 전체 로딩 화면을 보여주면 사용자 경험이 나빠집니다.">When fetching fresh data in the background while users are already viewing existing data, showing a full loading screen creates a poor user experience.</span>

**<span title="해결책">Solution:</span>** <span title="isPending과 isFetching을 구분하여 초기 로딩과 백그라운드 업데이트를 다르게 처리합니다.">Use `isPending` vs `isFetching` to handle initial loading differently from background updates.</span>

```tsx
function UserProfile({ userId }: { userId: string }) {
  const { data, isPending, isFetching, isSuccess } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000, // 5분
  })

  // 초기 로딩: 전체 로딩 화면
  if (isPending) {
    return <div>사용자 정보를 불러오는 중...</div>
  }

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
      {/* 백그라운드 업데이트: 작은 인디케이터만 표시 */}
      {isFetching && isSuccess && (
        <div className="refresh-indicator">🔄 업데이트 중...</div>
      )}
    </div>
  )
}
```

### <span title="2. 네트워크 상태 기반 사용자 피드백">2. Network-Aware User Feedback</span>

**<span title="문제 상황">Problem:</span>** <span title="네트워크 연결이 불안정한 환경에서 사용자는 왜 데이터가 로딩되지 않는지 알 수 없어 답답함을 느낍니다.">In unstable network conditions, users don't understand why data isn't loading, leading to frustration.</span>

**<span title="해결책">Solution:</span>** <span title="fetchStatus를 활용하여 네트워크 상태에 따른 구체적인 피드백을 제공합니다.">Use `fetchStatus` to provide specific feedback based on network conditions.</span>

```tsx
function TodoList() {
  const {
    data,
    isPending,
    isError,
    error,
    fetchStatus
  } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    retry: 3,
  })

  if (isPending && fetchStatus === 'fetching') {
    return <div>할 일 목록을 불러오는 중...</div>
  }

  // 네트워크 연결 문제를 명확히 알림
  if (isPending && fetchStatus === 'paused') {
    return (
      <div className="network-error">
        📶 네트워크 연결을 확인해주세요.
        연결되면 자동으로 데이터를 불러옵니다.
      </div>
    )
  }

  if (isError) {
    return (
      <div className="error-state">
        ❌ 오류: {error.message}
        {fetchStatus === 'fetching' && <span> (재시도 중...)</span>}
      </div>
    )
  }

  return (
    <ul>
      {data?.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```

### <span title="3. 실시간 데이터의 신선도 표시">3. Real-time Data Freshness Indicators</span>

**<span title="문제 상황">Problem:</span>** <span title="실시간 데이터를 다루는 애플리케이션에서 사용자는 현재 보고 있는 데이터가 얼마나 최신인지 알고 싶어합니다.">In real-time applications, users want to know how fresh the data they're viewing is.</span>

**<span title="해결책">Solution:</span>** <span title="isFetching, dataUpdatedAt 등을 조합하여 데이터의 실시간성을 시각적으로 표현합니다.">Combine `isFetching`, `dataUpdatedAt`, and other states to visually represent data freshness.</span>

```tsx
function StockPrice({ symbol }: { symbol: string }) {
  const {
    data,
    isSuccess,
    isFetching,
    dataUpdatedAt
  } = useQuery({
    queryKey: ['stock', symbol],
    queryFn: () => fetchStockPrice(symbol),
    refetchInterval: 1000, // 1초마다 업데이트
  })

  const formatTime = (timestamp: number) =>
    new Date(timestamp).toLocaleTimeString()

  return (
    <div className="stock-widget">
      <h3>{symbol}</h3>
      {isSuccess && (
        <>
          <div className="price">${data.price}</div>
          <div className="last-update">
            마지막 업데이트: {formatTime(dataUpdatedAt)}
            {/* 실시간 업데이트 표시 */}
            {isFetching && <span className="live-indicator">🔴 Live</span>}
          </div>
        </>
      )}
    </div>
  )
}
```

### <span title="4. 무한 스크롤의 복합 로딩 상태">4. Complex Loading States in Infinite Scrolling</span>

**<span title="문제 상황">Problem:</span>** <span title="무한 스크롤에서는 초기 로딩, 다음 페이지 로딩, 로딩 완료 상태가 모두 다른 UI를 필요로 합니다.">Infinite scrolling requires different UI for initial loading, next page loading, and completion states.</span>

**<span title="해결책">Solution:</span>** <span title="isPending, isFetchingNextPage, hasNextPage를 조합하여 각 상황에 맞는 UI를 제공합니다.">Combine `isPending`, `isFetchingNextPage`, and `hasNextPage` to provide appropriate UI for each scenario.</span>

```tsx
function InfinitePostList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isPending,
    isError,
    error
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam = 0 }) => fetchPosts(pageParam),
    getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
  })

  // 초기 로딩
  if (isPending) return <div>게시글을 불러오는 중...</div>
  if (isError) return <div>오류: {error.message}</div>

  return (
    <div>
      {data.pages.map((group, i) => (
        <div key={i}>
          {group.posts.map(post => (
            <article key={post.id}>
              <h3>{post.title}</h3>
              <p>{post.excerpt}</p>
            </article>
          ))}
        </div>
      ))}

      {/* 다음 페이지 로딩 상태를 명확히 구분 */}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? '더 불러오는 중...'
          : hasNextPage
          ? '더 보기'
          : '더 이상 없음'}
      </button>
    </div>
  )
}
```

### <span title="5. 종합적인 상태 관리 대시보드">5. Comprehensive State Management Dashboard</span>

**<span title="문제 상황">Problem:</span>** <span title="복잡한 대시보드에서는 데이터의 상태, 신선도, 에러 상황 등을 종합적으로 사용자에게 알려줘야 합니다.">Complex dashboards need to communicate data state, freshness, error conditions, and more comprehensively to users.</span>

**<span title="해결책">Solution:</span>** <span title="모든 상태값을 조합하여 세밀한 사용자 피드백을 구현합니다.">Combine all state values to implement granular user feedback.</span>

```tsx
function DataDashboard() {
  const {
    data,
    status,
    fetchStatus,
    isRefetching,
    isStale,
    errorUpdateCount
  } = useQuery({
    queryKey: ['dashboard'],
    queryFn: fetchDashboardData,
    staleTime: 2 * 60 * 1000, // 2분
    refetchOnWindowFocus: true,
  })

  // 복합적인 상태 해석
  const getStatusMessage = () => {
    if (status === 'pending' && fetchStatus === 'fetching') {
      return '대시보드 데이터 로딩 중...'
    }
    if (status === 'pending' && fetchStatus === 'paused') {
      return '네트워크 연결 대기 중...'
    }
    if (status === 'error') {
      return `오류 발생 (재시도 ${errorUpdateCount}회)`
    }
    if (status === 'success' && isRefetching) {
      return '데이터 새로고침 중...'
    }
    if (status === 'success' && isStale) {
      return '데이터가 오래됨 (새로고침 권장)'
    }
    return '최신 상태'
  }

  return (
    <div className="dashboard">
      <header>
        <h1>대시보드</h1>
        <div className={`status ${status}`}>
          {getStatusMessage()}
        </div>
      </header>

      {data && (
        <div className="metrics">
          <div className="metric">
            <h3>총 사용자</h3>
            <p>{data.totalUsers}</p>
          </div>
          <div className="metric">
            <h3>일일 활성 사용자</h3>
            <p>{data.dailyActiveUsers}</p>
          </div>
        </div>
      )}
    </div>
  )
}
```

### <span title="6. 지능적인 에러 복구 시스템">6. Intelligent Error Recovery System</span>

**<span title="문제 상황">Problem:</span>** <span title="네트워크 불안정이나 서버 문제로 인한 일시적 에러와 영구적 에러를 구분해서 처리해야 합니다.">Temporary errors due to network instability or server issues need to be handled differently from permanent errors.</span>

**<span title="해결책">Solution:</span>** <span title="failureCount, failureReason 등을 활용하여 지능적인 재시도와 에러 복구 로직을 구현합니다.">Use `failureCount`, `failureReason`, and other error states to implement intelligent retry and recovery logic.</span>

```tsx
function RobustDataFetcher() {
  const {
    data,
    isError,
    error,
    isPending,
    failureCount,
    failureReason
  } = useQuery({
    queryKey: ['critical-data'],
    queryFn: fetchCriticalData,
    retry: (failureCount, error) => {
      // 404 에러는 재시도하지 않음
      if (error.status === 404) return false
      return failureCount < 3
    },
    retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
  })

  if (isPending) {
    return (
      <div>
        {failureCount > 0 ? (
          <div>
            재시도 중... ({failureCount}/3)
            <br />
            <small>이전 오류: {failureReason?.message}</small>
          </div>
        ) : (
          <div>데이터 로딩 중...</div>
        )}
      </div>
    )
  }

  if (isError) {
    return (
      <div className="error-recovery">
        <p>❌ 최종 실패: {error.message}</p>
        <p>총 {failureCount}회 시도했습니다.</p>
        <button onClick={() => window.location.reload()}>
          페이지 새로고침
        </button>
      </div>
    )
  }

  return <div>데이터: {JSON.stringify(data, null, 2)}</div>
}
```

<span title="이러한 예시들을 통해 TanStack Query의 상태 시스템이 단순한 로딩 표시를 넘어서 사용자에게 정확하고 유용한 피드백을 제공하는 강력한 도구임을 알 수 있습니다.">Through these examples, we can see that TanStack Query's state system is a powerful tool that goes beyond simple loading indicators to provide accurate and useful feedback to users.</span>


## <span title="완전한 상태 레퍼런스">Complete State Reference</span>

<span title="useQuery가 반환하는 모든 상태값들과 그 의미를 체계적으로 정리했습니다.">Here's a comprehensive reference of all state values returned by `useQuery` and their meanings.</span>

### <span title="기본 상태 (Primary States)">Primary States</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `status` | `'pending' \| 'error' \| 'success'` | <span title="쿼리의 전반적인 상태를 나타냅니다. 데이터의 유무를 판단하는 주요 지표입니다.">Overall query state. Primary indicator of data availability.</span> |
| `fetchStatus` | `'fetching' \| 'paused' \| 'idle'` | <span title="queryFn의 실행 상태를 나타냅니다. 네트워크 요청의 진행 상황을 추적합니다.">Execution state of queryFn. Tracks network request progress.</span> |

### <span title="불리언 상태 지표 (Boolean State Indicators)">Boolean State Indicators</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `isPending` | `boolean` | <span title="status === 'pending'와 동일. 아직 데이터가 없는 상태입니다.">Equivalent to `status === 'pending'`. No data available yet.</span> |
| `isError` | `boolean` | <span title="status === 'error'와 동일. 쿼리 실행 중 오류가 발생한 상태입니다.">Equivalent to `status === 'error'`. Query encountered an error.</span> |
| `isSuccess` | `boolean` | <span title="status === 'success'와 동일. 쿼리가 성공적으로 완료되어 데이터가 있는 상태입니다.">Equivalent to `status === 'success'`. Query completed successfully with data.</span> |
| `isFetching` | `boolean` | <span title="fetchStatus === 'fetching'와 동일. 현재 쿼리가 실행 중인 상태입니다 (초기 로딩 또는 백그라운드 리페치 포함).">Equivalent to `fetchStatus === 'fetching'`. Query is currently executing (initial or background).</span> |
| `isRefetching` | `boolean` | <span title="데이터가 있는 상태에서 백그라운드로 새 데이터를 가져오는 중입니다.">Fetching fresh data in background while existing data is available.</span> |
| `isStale` | `boolean` | <span title="데이터가 staleTime을 초과하여 오래된 것으로 간주되는 상태입니다.">Data is considered stale (older than `staleTime`).</span> |
| `isPlaceholderData` | `boolean` | <span title="현재 표시되는 데이터가 placeholderData인 상태입니다.">Currently displayed data is `placeholderData`.</span> |
| `isLoadingError` | `boolean` | <span title="초기 로딩 중 에러가 발생한 상태입니다 (데이터가 없는 상태에서의 에러).">Error occurred during initial loading (no data available).</span> |
| `isRefetchError` | `boolean` | <span title="백그라운드 리페치 중 에러가 발생한 상태입니다 (기존 데이터는 유지됨).">Error occurred during background refetch (existing data remains).</span> |

### <span title="데이터 및 에러 정보 (Data and Error Information)">Data and Error Information</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `data` | `TData \| undefined` | <span title="쿼리에서 성공적으로 가져온 데이터입니다. isSuccess가 true일 때만 사용 가능합니다.">Successfully fetched data. Available when `isSuccess` is true.</span> |
| `error` | `TError \| null` | <span title="쿼리 실행 중 발생한 에러 객체입니다. isError가 true일 때 사용 가능합니다.">Error object from failed query. Available when `isError` is true.</span> |
| `dataUpdatedAt` | `number` | <span title="데이터가 마지막으로 성공적으로 업데이트된 시간의 타임스탬프입니다.">Timestamp of when data was last successfully updated.</span> |
| `errorUpdatedAt` | `number` | <span title="에러가 마지막으로 업데이트된 시간의 타임스탬프입니다.">Timestamp of when error was last updated.</span> |

### <span title="재시도 및 실패 정보 (Retry and Failure Information)">Retry and Failure Information</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `failureCount` | `number` | <span title="현재 쿼리 실행에서 발생한 연속된 실패 횟수입니다.">Number of consecutive failures for current query execution.</span> |
| `failureReason` | `TError \| null` | <span title="가장 최근 실패의 원인이 된 에러입니다. 재시도가 더 있을 경우에 유용합니다.">Error that caused the most recent failure. Useful during retry attempts.</span> |
| `errorUpdateCount` | `number` | <span title="이 쿼리에서 에러가 업데이트된 총 횟수입니다.">Total number of times error has been updated for this query.</span> |

### <span title="무한 쿼리 전용 상태 (Infinite Query Specific States)">Infinite Query Specific States</span>

<span title="useInfiniteQuery 사용 시에만 추가로 제공되는 상태들입니다.">Additional states available only when using `useInfiniteQuery`.</span>

| <span title="상태">State</span> | <span title="타입">Type</span> | <span title="설명">Description</span> |
|---------|------|-------------|
| `hasNextPage` | `boolean` | <span title="다음 페이지가 있는지 여부입니다. getNextPageParam의 반환값에 따라 결정됩니다.">Whether next page is available. Determined by `getNextPageParam` return value.</span> |
| `hasPreviousPage` | `boolean` | <span title="이전 페이지가 있는지 여부입니다. getPreviousPageParam의 반환값에 따라 결정됩니다.">Whether previous page is available. Determined by `getPreviousPageParam` return value.</span> |
| `isFetchingNextPage` | `boolean` | <span title="다음 페이지를 가져오는 중인지 여부입니다.">Whether next page is currently being fetched.</span> |
| `isFetchingPreviousPage` | `boolean` | <span title="이전 페이지를 가져오는 중인지 여부입니다.">Whether previous page is currently being fetched.</span> |

### <span title="상태 조합 패턴 (Common State Combination Patterns)">Common State Combination Patterns</span>

| <span title="시나리오">Scenario</span> | <span title="상태 조합">State Combination</span> | <span title="의미">Meaning</span> |
|----------|------------------|---------|
| <span title="초기 로딩">Initial Loading</span> | `isPending: true, isFetching: true` | <span title="첫 번째 데이터 로딩 중">Loading data for the first time</span> |
| <span title="백그라운드 리페치">Background Refetch</span> | `isSuccess: true, isFetching: true` | <span title="기존 데이터를 표시하면서 새 데이터 가져오는 중">Showing existing data while fetching fresh data</span> |
| <span title="네트워크 연결 대기">Network Paused</span> | `isPending: true, fetchStatus: 'paused'` | <span title="네트워크 연결을 기다리는 중">Waiting for network connection</span> |
| <span title="오래된 데이터 표시">Showing Stale Data</span> | `isSuccess: true, isStale: true` | <span title="데이터가 있지만 staleTime을 초과함">Data available but considered stale</span> |
| <span title="재시도 중">Retrying</span> | `isError: true, isFetching: true` | <span title="에러 발생 후 재시도 중">Retrying after an error occurred</span> |
| <span title="플레이스홀더 표시">Showing Placeholder</span> | `isPending: true, isPlaceholderData: true` | <span title="실제 데이터 로딩 중 임시 데이터 표시">Showing temporary data while loading real data</span> |

<span title="이 표를 참조하여 애플리케이션의 요구사항에 맞는 적절한 상태 조합을 선택할 수 있습니다.">Use this reference to select appropriate state combinations for your application's requirements.</span>

[//]: # 'Materials'

## <span title="추가 자료">Further Reading</span>

<span title="상태 체크의 대안적인 방법은 커뮤니티 리소스를 참고하세요.">For an alternative way of performing status checks, have a look at the [Community Resources](../../community/tkdodos-blog.md#4-status-checks-in-react-query).</span>

[//]: # 'Materials'

## 요약

- TanStack Query의 쿼리는 고유한 queryKey와 비동기 queryFn을 기반으로 서버 데이터를 패칭, 캐싱, 동기화, 업데이트합니다.
- 쿼리의 상태는 pending, error, success로 구분되며, fetchStatus(예: fetching, paused, idle)로 네트워크/백그라운드 상태를 세밀하게 파악할 수 있습니다.
- isPending, isError, isSuccess, isFetching 등 다양한 상태값을 활용해 UI를 직관적으로 제어할 수 있습니다.
- 쿼리의 상태와 fetchStatus를 조합해, 데이터의 유무와 네트워크 동작을 명확히 구분할 수 있습니다.
- 대부분의 실전 상황에서는 isPending → isError → 성공 순으로 분기 처리하면 충분하며, 필요에 따라 status/fetchStatus를 세밀하게 활용할 수 있습니다.
