---
id: query-options
title: Query Options
---

<span title="여러 곳에서 queryKey와 queryFn을 공유하면서도 서로 가까이 둘 수 있는 최고의 방법 중 하나는 queryOptions 헬퍼를 사용하는 것입니다. 런타임에는 이 헬퍼가 전달받은 값을 그대로 반환하지만, TypeScript와 함께 사용할 때 많은 장점이 있습니다. 쿼리의 모든 옵션을 한 곳에 정의할 수 있고, 타입 추론과 타입 안전성도 모두 보장받을 수 있습니다.">One of the best ways to share `queryKey` and `queryFn` between multiple places, yet keep them co-located to one another, is to use the `queryOptions` helper. At runtime, this helper just returns whatever you pass into it, but it has a lot of advantages when using it [with TypeScript](../../typescript.md#typing-query-options). You can define all possible options for a query in one place, and you'll also get type inference and type safety for all of them.</span>

[//]: # 'Example1'

```ts
import { queryOptions } from '@tanstack/react-query'

function groupOptions(id: number) {
  return queryOptions({
    queryKey: ['groups', id],
    queryFn: () => fetchGroups(id),
    staleTime: 5 * 1000,
  })
}

// usage:

useQuery(groupOptions(1))
useSuspenseQuery(groupOptions(5))
useQueries({
  queries: [groupOptions(1), groupOptions(2)],
})
queryClient.prefetchQuery(groupOptions(23))
queryClient.setQueryData(groupOptions(42).queryKey, newGroups)
```

[//]: # 'Example1'

<span title="Infinite Query의 경우 별도의 infiniteQueryOptions 헬퍼가 제공됩니다.">For Infinite Queries, a separate [`infiniteQueryOptions`](../../reference/infiniteQueryOptions.md) helper is available.</span>

<span title="컴포넌트 레벨에서 일부 옵션을 오버라이드할 수도 있습니다. 매우 흔하고 유용한 패턴은 컴포넌트별 select 함수를 만드는 것입니다.">You can still override some options at the component level. A very common and useful pattern is to create per-component [`select`](../render-optimizations.md#select) functions:</span>

[//]: # 'Example2'

```ts
// Type inference still works, so query.data will be the return type of select instead of queryFn

const query = useQuery({
  ...groupOptions(1),
  select: (data) => data.groupName,
})
```

[//]: # 'Example2'

## 요약

- queryOptions 헬퍼를 사용하면 queryKey와 queryFn을 여러 곳에서 재사용하면서도, 타입 추론과 타입 안전성을 유지할 수 있습니다.
- 쿼리의 모든 옵션을 한 곳에 정의하고, 필요에 따라 컴포넌트 레벨에서 select 등 일부 옵션만 오버라이드하는 패턴이 실전에서 매우 유용합니다.
- Infinite Query의 경우 별도의 infiniteQueryOptions 헬퍼를 활용할 수 있습니다.
- queryOptions 패턴은 코드의 일관성, 재사용성, 유지보수성을 크게 높여주며, 타입스크립트 환경에서 특히 강력한 이점을 제공합니다.
