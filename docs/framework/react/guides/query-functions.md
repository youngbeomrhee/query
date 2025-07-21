---
id: query-functions
title: Query Functions
---

<span title="쿼리 함수는 실제로 Promise를 반환하는 어떤 함수든 될 수 있습니다. 반환된 Promise는 데이터를 resolve하거나 에러를 throw해야 합니다.">A query function can be literally any function that **returns a promise**. The promise that is returned should either **resolve the data** or **throw an error**.</span>

<span title="아래 모든 예시는 유효한 쿼리 함수 구성입니다.">All of the following are valid query function configurations:</span>

[//]: # 'Example'

```tsx
useQuery({ queryKey: ['todos'], queryFn: fetchAllTodos })
useQuery({ queryKey: ['todos', todoId], queryFn: () => fetchTodoById(todoId) })
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    const data = await fetchTodoById(todoId)
    return data
  },
})
useQuery({
  queryKey: ['todos', todoId],
  queryFn: ({ queryKey }) => fetchTodoById(queryKey[1]),
})
```

[//]: # 'Example'

## <span title="에러 처리 및 throw">Handling and Throwing Errors</span>

<span title="TanStack Query가 쿼리에서 에러가 발생했는지 판단하려면, 쿼리 함수가 반드시 에러를 throw하거나 rejected Promise를 반환해야 합니다. 쿼리 함수에서 throw된 에러는 쿼리의 error 상태에 저장됩니다.">For TanStack Query to determine a query has errored, the query function **must throw** or return a **rejected Promise**. Any error that is thrown in the query function will be persisted on the `error` state of the query.</span>

[//]: # 'Example2'

```tsx
const { error } = useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    if (somethingGoesWrong) {
      throw new Error('Oh no!')
    }
    if (somethingElseGoesWrong) {
      return Promise.reject(new Error('Oh no!'))
    }

    return data
  },
})
```

[//]: # 'Example2'

## <span title="기본적으로 에러를 throw하지 않는 fetch 및 기타 클라이언트와의 사용">Usage with `fetch` and other clients that do not throw by default</span>

<span title="axios나 graphql-request 같은 대부분의 유틸리티는 HTTP 호출이 실패하면 자동으로 에러를 throw하지만, fetch 같은 일부 유틸리티는 기본적으로 에러를 throw하지 않습니다. 이 경우 직접 throw해야 합니다. 아래는 fetch API로 처리하는 예시입니다.">While most utilities like `axios` or `graphql-request` automatically throw errors for unsuccessful HTTP calls, some utilities like `fetch` do not throw errors by default. If that's the case, you'll need to throw them on your own. Here is a simple way to do that with the popular `fetch` API:</span>

[//]: # 'Example3'

```tsx
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    const response = await fetch('/todos/' + todoId)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    return response.json()
  },
})
```

[//]: # 'Example3'

## <span title="쿼리 함수 변수">Query Function Variables</span>

<span title="쿼리 키는 단순히 가져올 데이터를 고유하게 식별하는 용도일 뿐만 아니라, QueryFunctionContext의 일부로 쿼리 함수에 전달됩니다. 항상 필요한 것은 아니지만, 쿼리 함수를 분리할 때 유용합니다.">Query keys are not just for uniquely identifying the data you are fetching, but are also conveniently passed into your query function as part of the QueryFunctionContext. While not always necessary, this makes it possible to extract your query functions if needed:</span>

[//]: # 'Example4'

```tsx
function Todos({ status, page }) {
  const result = useQuery({
    queryKey: ['todos', { status, page }],
    queryFn: fetchTodoList,
  })
}

// Access the key, status and page variables in your query function!
function fetchTodoList({ queryKey }) {
  const [_key, { status, page }] = queryKey
  return new Promise()
}
```

[//]: # 'Example4'

### <span title="QueryFunctionContext">QueryFunctionContext</span>

<span title="QueryFunctionContext는 각 쿼리 함수에 전달되는 객체입니다. 구성 요소는 다음과 같습니다:">The `QueryFunctionContext` is the object passed to each query function. It consists of:</span>

- <span title="queryKey: 쿼리 키">`queryKey: QueryKey`: [Query Keys](../query-keys.md)</span>
- <span title="client: QueryClient 인스턴스">`client: QueryClient`: [QueryClient](../../../../reference/QueryClient.md)</span>
- <span title="signal?: AbortSignal (옵션)">`signal?: AbortSignal`</span>
  - <span title="TanStack Query가 제공하는 AbortSignal 인스턴스">[AbortSignal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) instance provided by TanStack Query</span>
  - <span title="쿼리 취소에 사용할 수 있습니다.">Can be used for [Query Cancellation](../query-cancellation.md)</span>
- <span title="meta: Record<string, unknown> | undefined (옵션)">`meta: Record<string, unknown> | undefined`</span>
  - <span title="쿼리에 추가 정보를 채울 수 있는 선택적 필드">an optional field you can fill with additional information about your query</span>

<span title="추가로, Infinite Queries는 다음 옵션도 전달받습니다:">Additionally, [Infinite Queries](../infinite-queries.md) get the following options passed:</span>

- <span title="pageParam: 현재 페이지를 가져오는 데 사용되는 파라미터">`pageParam: TPageParam`</span>
  - <span title="현재 페이지 fetch에 사용되는 페이지 파라미터">the page parameter used to fetch the current page</span>
- <span title="direction: 'forward' | 'backward' (deprecated)">`direction: 'forward' | 'backward'`</span>
  - <span title="현재 페이지 fetch의 방향 (deprecated)">**deprecated**</span>
  - <span title="현재 페이지 fetch의 방향. direction이 필요하다면 getNextPageParam/getPreviousPageParam에서 pageParam에 직접 추가하세요.">the direction of the current page fetch. To get access to the direction of the current page fetch, please add a direction to `pageParam` from `getNextPageParam` and `getPreviousPageParam`.</span>

## 요약

- 쿼리 함수는 Promise를 반환하는 어떤 함수든 가능하며, 데이터 resolve 또는 에러 throw가 필수입니다.
- 쿼리 함수에서 에러를 throw하거나 rejected Promise를 반환해야 쿼리의 error 상태가 올바르게 동작합니다.
- fetch, axios 등 다양한 클라이언트에서 에러 처리를 명확히 해야 하며, fetch는 직접 throw가 필요합니다.
- 쿼리 키(queryKey)는 쿼리 함수에 QueryFunctionContext로 전달되어, 동적 파라미터 처리와 함수 분리에 유용하게 활용할 수 있습니다.
- QueryFunctionContext에는 queryKey, client, signal, meta 등이 포함되며, Infinite Query의 경우 pageParam 등 추가 옵션도 전달됩니다.
