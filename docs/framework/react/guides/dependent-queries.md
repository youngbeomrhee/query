---
id: dependent-queries
title: Dependent Queries
---

## <span title="useQuery 의존 쿼리">useQuery dependent Query</span>

<span title="의존(또는 직렬) 쿼리는 이전 쿼리가 끝나야 실행할 수 있습니다. 이를 위해 enabled 옵션을 사용해 쿼리가 언제 실행 준비가 되었는지 지정하면 됩니다.">Dependent (or serial) queries depend on previous ones to finish before they can execute. To achieve this, it's as easy as using the `enabled` option to tell a query when it is ready to run:</span>

[//]: # 'Example'

```tsx
// Get the user
const { data: user } = useQuery({
  queryKey: ['user', email],
  queryFn: getUserByEmail,
})

const userId = user?.id

// Then get the user's projects
const {
  status,
  fetchStatus,
  data: projects,
} = useQuery({
  queryKey: ['projects', userId],
  queryFn: getProjectsByUser,
  // The query will not execute until the userId exists
  enabled: !!userId,
})
```

[//]: # 'Example'

<span title="projects 쿼리는 다음 상태로 시작합니다:">The `projects` query will start in:</span>

```tsx
status: 'pending'
isPending: true
fetchStatus: 'idle'
```

<span title="user가 준비되면 projects 쿼리가 enabled되어 다음 상태로 전환됩니다:">As soon as the `user` is available, the `projects` query will be `enabled` and will then transition to:</span>

```tsx
status: 'pending'
isPending: true
fetchStatus: 'fetching'
```

<span title="프로젝트를 가져오면 다음 상태가 됩니다:">Once we have the projects, it will go to:</span>

```tsx
status: 'success'
isPending: false
fetchStatus: 'idle'
```

## <span title="useQueries 의존 쿼리">useQueries dependent Query</span>

<span title="동적 병렬 쿼리 - useQueries도 이전 쿼리에 의존할 수 있습니다. 방법은 다음과 같습니다:">Dynamic parallel query - `useQueries` can depend on a previous query also, here's how to achieve this:</span>

[//]: # 'Example2'

```tsx
// Get the users ids
const { data: userIds } = useQuery({
  queryKey: ['users'],
  queryFn: getUsersData,
  select: (users) => users.map((user) => user.id),
})

// Then get the users messages
const usersMessages = useQueries({
  queries: userIds
    ? userIds.map((id) => {
        return {
          queryKey: ['messages', id],
          queryFn: () => getMessagesByUsers(id),
        }
      })
    : [], // if userIds is undefined, an empty array will be returned
})
```

[//]: # 'Example2'

<span title="useQueries는 쿼리 결과 배열을 반환한다는 점에 유의하세요."><strong>Note</strong> that `useQueries` return an <strong>array of query results</strong></span>

## <span title="성능에 대한 참고">A note about performance</span>

<span title="의존 쿼리는 본질적으로 request waterfall(요청 폭포수) 형태를 띠므로 성능에 불리합니다. 두 쿼리의 시간이 같다고 가정하면, 직렬로 처리하면 항상 두 배의 시간이 걸리며, 특히 지연이 큰 클라이언트에서 더 치명적입니다. 가능하다면 백엔드 API를 병렬로 쿼리할 수 있도록 재구성하는 것이 좋지만, 항상 실현 가능한 것은 아닙니다.">Dependent queries by definition constitutes a form of [request waterfall](../request-waterfalls.md), which hurts performance. If we pretend both queries take the same amount of time, doing them serially instead of in parallel always takes twice as much time, which is especially hurtful when it happens on a client that has high latency. If you can, it's always better to restructure the backend APIs so that both queries can be fetched in parallel, though that might not always be practically feasible.</span>

<span title="위 예시에서 getUserByEmail로 먼저 가져온 뒤 getProjectsByUser를 호출하는 대신, getProjectsByUserEmail 쿼리를 새로 만들어 waterfall을 평탄화할 수 있습니다.">In the example above, instead of first fetching `getUserByEmail` to be able to `getProjectsByUser`, introducing a new `getProjectsByUserEmail` query would flatten the waterfall.</span>

## 요약

- 의존(직렬) 쿼리는 이전 쿼리의 결과가 준비된 후에만 실행되며, enabled 옵션을 활용해 쉽게 구현할 수 있습니다.
- useQueries 훅을 사용하면 동적 병렬 쿼리도 이전 쿼리의 결과(예: id 리스트)에 의존해 안전하게 실행할 수 있습니다.
- useQueries는 항상 쿼리 결과 배열을 반환하므로, 각 쿼리의 상태와 데이터를 개별적으로 관리할 수 있습니다.
- 의존 쿼리는 본질적으로 request waterfall(요청 폭포수) 구조이므로, 병렬 쿼리보다 느릴 수 있습니다. 가능하다면 백엔드 API를 병렬로 호출할 수 있도록 설계하는 것이 성능에 유리합니다.
- waterfall을 평탄화하려면, 여러 쿼리를 하나의 API로 합치거나, 의존성을 줄이는 쿼리 구조를 고민하는 것이 좋습니다.
