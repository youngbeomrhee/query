---
id: background-fetching-indicators
title: Background Fetching Indicators
---

<span title="쿼리의 status === 'pending' 상태만으로도 쿼리의 초기 하드 로딩 상태를 표시하기에 충분하지만, 때로는 쿼리가 백그라운드에서 refetch 중임을 추가로 표시하고 싶을 수 있습니다. 이를 위해 쿼리는 isFetching 불리언도 제공하며, status 변수와 상관없이 쿼리가 fetch 중임을 표시할 수 있습니다.">A query's `status === 'pending'` state is sufficient enough to show the initial hard-loading state for a query, but sometimes you may want to display an additional indicator that a query is refetching in the background. To do this, queries also supply you with an `isFetching` boolean that you can use to show that it's in a fetching state, regardless of the state of the `status` variable:</span>

[//]: # 'Example'

```tsx
function Todos() {
  const {
    status,
    data: todos,
    error,
    isFetching,
  } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })

  return status === 'pending' ? (
    <span>Loading...</span>
  ) : status === 'error' ? (
    <span>Error: {error.message}</span>
  ) : (
    <>
      {isFetching ? <div>Refreshing...</div> : null}

      <div>
        {todos.map((todo) => (
          <Todo todo={todo} />
        ))}
      </div>
    </>
  )
}
```

[//]: # 'Example'

## <span title="글로벌 백그라운드 패칭 로딩 상태 표시">Displaying Global Background Fetching Loading State</span>

<span title="개별 쿼리 로딩 상태 외에도, 모든 쿼리(백그라운드 포함)가 fetch 중일 때 글로벌 로딩 인디케이터를 표시하고 싶다면 useIsFetching 훅을 사용할 수 있습니다.">In addition to individual query loading states, if you would like to show a global loading indicator when **any** queries are fetching (including in the background), you can use the `useIsFetching` hook:</span>

[//]: # 'Example2'

```tsx
import { useIsFetching } from '@tanstack/react-query'

function GlobalLoadingIndicator() {
  const isFetching = useIsFetching()

  return isFetching ? (
    <div>Queries are fetching in the background...</div>
  ) : null
}
```

[//]: # 'Example2'

## 요약

- 쿼리의 status === 'pending' 상태는 초기 로딩을 표시하는 데 충분하지만, isFetching을 활용하면 백그라운드 refetch 등 다양한 fetch 상황을 세밀하게 감지할 수 있습니다.
- isFetching은 쿼리의 상태와 무관하게 fetch가 진행 중임을 알려주므로, UI에서 별도의 로딩/새로고침 인디케이터를 쉽게 구현할 수 있습니다.
- useIsFetching 훅을 사용하면 전체 앱에서 하나라도 쿼리가 fetch 중일 때 글로벌 로딩 인디케이터를 표시할 수 있습니다.
- 개별 쿼리와 글로벌 쿼리 fetch 상태를 분리해 관리하면, 사용자 경험을 더욱 세밀하게 제어할 수 있습니다.
