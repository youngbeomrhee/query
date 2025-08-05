---
id: invalidations-from-mutations
title: Invalidations from Mutations
---

Invalidating queries is only half the battle. Knowing **when** to invalidate them is the other half. Usually when a mutation in your app succeeds, it's VERY likely that there are related queries in your application that need to be invalidated and possibly refetched to account for the new changes from your mutation.

For example, assume we have a mutation to post a new todo:

[//]: # 'Example'

```tsx
const mutation = useMutation({ mutationFn: postTodo })
```

[//]: # 'Example'

When a successful `postTodo` mutation happens, we likely want all `todos` queries to get invalidated and possibly refetched to show the new todo item. To do this, you can use `useMutation`'s `onSuccess` options and the `client`'s `invalidateQueries` function:

[//]: # 'Example2'

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

const queryClient = useQueryClient()

// When this mutation succeeds, invalidate any queries with the `todos` or `reminders` query key
const mutation = useMutation({
  mutationFn: addTodo,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
    queryClient.invalidateQueries({ queryKey: ['reminders'] })
  },
})
```

[//]: # 'Example2'

You can wire up your invalidations to happen using any of the callbacks available in the [`useMutation` hook](../mutations.md)

## 요약

TanStack Query에서 뮤테이션이 성공적으로 완료되면 관련된 쿼리를 무효화하고 다시 가져와야 할 필요가 있습니다. 주요 포인트는 다음과 같습니다:

- **뮤테이션 후 무효화**: 뮤테이션이 성공하면 관련 쿼리를 무효화하여 최신 데이터를 반영할 수 있습니다.
- **`onSuccess` 콜백 사용**: `useMutation`의 `onSuccess` 옵션을 사용하여 뮤테이션 성공 시 특정 쿼리를 무효화할 수 있습니다.
- **`invalidateQueries` 함수**: `queryClient.invalidateQueries`를 사용하여 특정 쿼리 키를 가진 쿼리를 무효화할 수 있습니다.

이러한 방법을 통해 뮤테이션 후 데이터의 일관성을 유지하고, 사용자에게 최신 정보를 제공할 수 있습니다.
