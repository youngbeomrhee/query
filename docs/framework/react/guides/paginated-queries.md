---
id: paginated-queries
title: Paginated / Lagged Queries
---

Rendering paginated data is a very common UI pattern and in TanStack Query, it "just works" by including the page information in the query key:

[//]: # 'Example'

```tsx
const result = useQuery({
  queryKey: ['projects', page],
  queryFn: fetchProjects,
})
```

[//]: # 'Example'

However, if you run this simple example, you might notice something strange:

**The UI jumps in and out of the `success` and `pending` states because each new page is treated like a brand new query.**

This experience is not optimal and unfortunately is how many tools today insist on working. But not TanStack Query! As you may have guessed, TanStack Query comes with an awesome feature called `placeholderData` that allows us to get around this.

## Better Paginated Queries with `placeholderData`

Consider the following example where we would ideally want to increment a pageIndex (or cursor) for a query. If we were to use `useQuery`, **it would still technically work fine**, but the UI would jump in and out of the `success` and `pending` states as different queries are created and destroyed for each page or cursor. By setting `placeholderData` to `(previousData) => previousData` or `keepPreviousData` function exported from TanStack Query, we get a few new things:

- **The data from the last successful fetch is available while new data is being requested, even though the query key has changed**.
- When the new data arrives, the previous `data` is seamlessly swapped to show the new data.
- `isPlaceholderData` is made available to know what data the query is currently providing you

[//]: # 'Example2'

```tsx
import { keepPreviousData, useQuery } from '@tanstack/react-query'
import React from 'react'

function Todos() {
  const [page, setPage] = React.useState(0)

  const fetchProjects = (page = 0) =>
    fetch('/api/projects?page=' + page).then((res) => res.json())

  const { isPending, isError, error, data, isFetching, isPlaceholderData } =
    useQuery({
      queryKey: ['projects', page],
      queryFn: () => fetchProjects(page),
      placeholderData: keepPreviousData,
    })

  return (
    <div>
      {isPending ? (
        <div>Loading...</div>
      ) : isError ? (
        <div>Error: {error.message}</div>
      ) : (
        <div>
          {data.projects.map((project) => (
            <p key={project.id}>{project.name}</p>
          ))}
        </div>
      )}
      <span>Current Page: {page + 1}</span>
      <button
        onClick={() => setPage((old) => Math.max(old - 1, 0))}
        disabled={page === 0}
      >
        Previous Page
      </button>
      <button
        onClick={() => {
          if (!isPlaceholderData && data.hasMore) {
            setPage((old) => old + 1)
          }
        }}
        // Disable the Next Page button until we know a next page is available
        disabled={isPlaceholderData || !data?.hasMore}
      >
        Next Page
      </button>
      {isFetching ? <span> Loading...</span> : null}
    </div>
  )
}
```

[//]: # 'Example2'

## Lagging Infinite Query results with `placeholderData`

While not as common, the `placeholderData` option also works flawlessly with the `useInfiniteQuery` hook, so you can seamlessly allow your users to continue to see cached data while infinite query keys change over time.

## 요약

TanStack Query는 페이지네이션된 데이터를 렌더링할 때, 쿼리 키에 페이지 정보를 포함시켜 간단하게 처리할 수 있습니다. 그러나 각 페이지가 새로운 쿼리로 처리되기 때문에 UI가 `success`와 `pending` 상태를 반복적으로 전환할 수 있습니다. 이를 해결하기 위해 `placeholderData`를 사용하여 다음과 같은 이점을 얻을 수 있습니다:

- 마지막으로 성공한 데이터가 새로운 데이터를 요청하는 동안에도 사용 가능합니다.
- 새로운 데이터가 도착하면 이전 데이터가 매끄럽게 교체됩니다.
- `isPlaceholderData`를 통해 현재 제공되는 데이터의 상태를 알 수 있습니다.

또한, `placeholderData`는 `useInfiniteQuery` 훅과도 잘 작동하여, 무한 스크롤 쿼리 키가 시간에 따라 변경될 때도 사용자가 캐시된 데이터를 계속 볼 수 있도록 합니다.
