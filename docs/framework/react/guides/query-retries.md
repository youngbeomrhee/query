---
id: query-retries
title: Query Retries
---

When a `useQuery` query fails (the query function throws an error), TanStack Query will automatically retry the query if that query's request has not reached the max number of consecutive retries (defaults to `3`) or a function is provided to determine if a retry is allowed.

You can configure retries both on a global level and an individual query level.

- Setting `retry = false` will disable retries.
- Setting `retry = 6` will retry failing requests 6 times before showing the final error thrown by the function.
- Setting `retry = true` will infinitely retry failing requests.
- Setting `retry = (failureCount, error) => ...` allows for custom logic based on why the request failed.

[//]: # 'Info'

> On the server, retries default to `0` to make server rendering as fast as possible.

[//]: # 'Info'
[//]: # 'Example'

```tsx
import { useQuery } from '@tanstack/react-query'

// Make a specific query retry a certain number of times
const result = useQuery({
  queryKey: ['todos', 1],
  queryFn: fetchTodoListPage,
  retry: 10, // Will retry failed requests 10 times before displaying an error
})
```

[//]: # 'Example'

> Info: Contents of the `error` property will be part of `failureReason` response property of `useQuery` until the last retry attempt. So in above example any error contents will be part of `failureReason` property for first 9 retry attempts (Overall 10 attempts) and finally they will be part of `error` after last attempt if error persists after all retry attempts.

## Retry Delay

By default, retries in TanStack Query do not happen immediately after a request fails. As is standard, a back-off delay is gradually applied to each retry attempt.

The default `retryDelay` is set to double (starting at `1000`ms) with each attempt, but not exceed 30 seconds:

[//]: # 'Example2'

```tsx
// Configure for all queries
import {
  QueryCache,
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
})

function App() {
  return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
}
```

[//]: # 'Example2'

Though it is not recommended, you can obviously override the `retryDelay` function/integer in both the Provider and individual query options. If set to an integer instead of a function the delay will always be the same amount of time:

[//]: # 'Example3'

```tsx
const result = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
  retryDelay: 1000, // Will always wait 1000ms to retry, regardless of how many retries
})
```

[//]: # 'Example3'

## 요약

TanStack Query는 쿼리가 실패할 경우 자동으로 재시도를 수행합니다. 기본적으로 최대 3번의 연속된 재시도를 허용하며, 재시도 횟수는 전역 또는 개별 쿼리 수준에서 설정할 수 있습니다.

- `retry = false`: 재시도를 비활성화합니다.
- `retry = 6`: 실패한 요청을 6번 재시도합니다.
- `retry = true`: 실패한 요청을 무한히 재시도합니다.
- `retry = (failureCount, error) => ...`: 실패 원인에 따라 재시도 여부를 결정하는 사용자 정의 로직을 설정할 수 있습니다.

기본적으로 재시도는 즉시 발생하지 않으며, 각 시도마다 점진적으로 백오프 지연이 적용됩니다. 기본 `retryDelay`는 1000ms에서 시작하여 각 시도마다 두 배로 증가하며, 최대 30초를 초과하지 않습니다. 이 설정은 전역 및 개별 쿼리 옵션에서 재정의할 수 있습니다.
