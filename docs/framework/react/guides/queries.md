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
