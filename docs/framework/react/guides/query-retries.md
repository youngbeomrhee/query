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

- 쿼리 함수가 실패하면 TanStack Query는 기본적으로 최대 3회까지 자동 재시도를 수행합니다(서버 환경은 기본 0회).
- retry 옵션은 false(비활성화), true(무한 재시도), 숫자(정해진 횟수), 함수(실패 원인별 커스텀) 등 다양한 방식으로 설정할 수 있습니다.
- retryDelay는 기본적으로 1000ms에서 시작해 2배씩 증가(최대 30초)하는 backoff 전략을 사용하며, 숫자나 함수로 커스터마이즈 가능합니다.
- 재시도 중 발생한 에러는 failureReason에 저장되고, 마지막 시도 후에는 error로 노출됩니다.
- 전역/개별 쿼리 단위로 retry 및 retryDelay를 유연하게 설정할 수 있습니다.
