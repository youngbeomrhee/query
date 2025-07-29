---
id: placeholder-query-data
title: Placeholder Query Data
---

## What is placeholder data?

Placeholder data allows a query to behave as if it already has data, similar to the `initialData` option, but **the data is not persisted to the cache**. This comes in handy for situations where you have enough partial (or fake) data to render the query successfully while the actual data is fetched in the background.

> Example: An individual blog post query could pull "preview" data from a parent list of blog posts that only include title and a small snippet of the post body. You would not want to persist this partial data to the query result of the individual query, but it is useful for showing the content layout as quickly as possible while the actual query finishes to fetch the entire object.

There are a few ways to supply placeholder data for a query to the cache before you need it:

- Declaratively:
  - Provide `placeholderData` to a query to prepopulate its cache if empty
- Imperatively:
  - [Prefetch or fetch the data using `queryClient` and the `placeholderData` option](../prefetching.md)

When we use `placeholderData`, our Query will not be in a `pending` state - it will start out as being in `success` state, because we have `data` to display - even if that data is just "placeholder" data. To distinguish it from "real" data, we will also have the `isPlaceholderData` flag set to `true` on the Query result.

## Placeholder Data as a Value

[//]: # 'ExampleValue'

```tsx
function Todos() {
  const result = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/todos'),
    placeholderData: placeholderTodos,
  })
}
```

[//]: # 'ExampleValue'
[//]: # 'Memoization'

### Placeholder Data Memoization

If the process for accessing a query's placeholder data is intensive or just not something you want to perform on every render, you can memoize the value:

```tsx
function Todos() {
  const placeholderData = useMemo(() => generateFakeTodos(), [])
  const result = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/todos'),
    placeholderData,
  })
}
```

[//]: # 'Memoization'

## Placeholder Data as a Function

`placeholderData` can also be a function, where you can get access to the data and Query meta information of a "previous" successful Query. This is useful for situations where you want to use the data from one query as the placeholder data for another query. When the QueryKey changes, e.g. from `['todos', 1]` to `['todos', 2]`, we can keep displaying "old" data instead of having to show a loading spinner while data is _transitioning_ from one Query to the next. For more information, see [Paginated Queries](../paginated-queries.md).

[//]: # 'ExampleFunction'

```tsx
const result = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetch(`/todos/${id}`),
  placeholderData: (previousData, previousQuery) => previousData,
})
```

[//]: # 'ExampleFunction'

### Placeholder Data from Cache

In some circumstances, you may be able to provide the placeholder data for a query from the cached result of another. A good example of this would be searching the cached data from a blog post list query for a preview version of the post, then using that as the placeholder data for your individual post query:

[//]: # 'ExampleCache'

```tsx
function Todo({ blogPostId }) {
  const queryClient = useQueryClient()
  const result = useQuery({
    queryKey: ['blogPost', blogPostId],
    queryFn: () => fetch(`/blogPosts/${blogPostId}`),
    placeholderData: () => {
      // Use the smaller/preview version of the blogPost from the 'blogPosts'
      // query as the placeholder data for this blogPost query
      return queryClient
        .getQueryData(['blogPosts'])
        ?.find((d) => d.id === blogPostId)
    },
  })
}
```

[//]: # 'ExampleCache'
[//]: # 'Materials'

## Further reading

For a comparison between `Placeholder Data` and `Initial Data`, have a look at the [Community Resources](../../community/tkdodos-blog.md#9-placeholder-and-initial-data-in-react-query).

[//]: # 'Materials'

## 요약

- placeholderData는 쿼리의 초기 데이터처럼 동작하지만, 실제 캐시에 저장되지 않고 임시로만 사용됩니다.
- 부분 데이터, 프리뷰, 페이크 데이터 등 실제 데이터가 오기 전 UI를 빠르게 렌더링할 때 유용합니다.
- placeholderData는 값, 함수, 캐시 참조 등 다양한 방식으로 지정할 수 있으며, isPlaceholderData 플래그로 구분할 수 있습니다.
- 함수형 placeholderData를 활용하면 이전 쿼리의 데이터를 활용해 페이지 전환 시 로딩 스피너 없이 자연스러운 UX를 구현할 수 있습니다.
- 캐시에서 데이터를 가져와 placeholder로 활용할 때는, 실제 데이터가 오면 자동으로 대체됩니다.
- placeholderData와 initialData의 차이점은, placeholderData는 임시/프리뷰 용도이고, initialData는 실제 캐시에 저장된다는 점입니다.
