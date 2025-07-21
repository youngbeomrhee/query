---
id: parallel-queries
title: Parallel Queries
---

<span title="'병렬' 쿼리는 동시에 실행되어 데이터 패칭의 동시성을 극대화하는 쿼리입니다.">"Parallel" queries are queries that are executed in parallel, or at the same time so as to maximize fetching concurrency.</span>

## <span title="수동 병렬 쿼리">Manual Parallel Queries</span>

<span title="병렬 쿼리의 개수가 변하지 않는다면, 별도의 추가 작업 없이 여러 개의 useQuery나 useInfiniteQuery 훅을 나란히 사용하면 됩니다.">When the number of parallel queries does not change, there is **no extra effort** to use parallel queries. Just use any number of TanStack Query's `useQuery` and `useInfiniteQuery` hooks side-by-side!</span>

[//]: # 'Example'

```tsx
function App () {
  // The following queries will execute in parallel
  const usersQuery = useQuery({ queryKey: ['users'], queryFn: fetchUsers })
  const teamsQuery = useQuery({ queryKey: ['teams'], queryFn: fetchTeams })
  const projectsQuery = useQuery({ queryKey: ['projects'], queryFn: fetchProjects })
  ...
}
```

[//]: # 'Example'
[//]: # 'Info'

> <span title="React Query를 suspense 모드에서 사용할 때는 이 병렬 패턴이 동작하지 않습니다. 첫 번째 쿼리가 내부적으로 promise를 throw해서 컴포넌트를 suspend시키기 때문입니다. 이를 해결하려면 useSuspenseQueries 훅(권장)을 사용하거나, 각 useSuspenseQuery 인스턴스를 별도 컴포넌트로 분리해 직접 병렬 처리를 orchestrate해야 합니다.">When using React Query in suspense mode, this pattern of parallelism does not work, since the first query would throw a promise internally and would suspend the component before the other queries run. To get around this, you'll either need to use the `useSuspenseQueries` hook (which is suggested) or orchestrate your own parallelism with separate components for each `useSuspenseQuery` instance.</span>

[//]: # 'Info'

## <span title="useQueries를 활용한 동적 병렬 쿼리">Dynamic Parallel Queries with `useQueries`</span>

[//]: # 'DynamicParallelIntro'

<span title="실행해야 할 쿼리의 개수가 렌더마다 달라진다면, 수동 쿼리 방식은 훅의 규칙을 위반하므로 사용할 수 없습니다. 대신 TanStack Query의 useQueries 훅을 사용하면 원하는 만큼 동적으로 병렬 쿼리를 실행할 수 있습니다.">If the number of queries you need to execute is changing from render to render, you cannot use manual querying since that would violate the rules of hooks. Instead, TanStack Query provides a `useQueries` hook, which you can use to dynamically execute as many queries in parallel as you'd like.</span>

[//]: # 'DynamicParallelIntro'

<span title="useQueries는 queries 키에 쿼리 객체 배열을 담은 옵션 객체를 받고, 쿼리 결과 배열을 반환합니다.">`useQueries` accepts an **options object** with a **queries key** whose value is an **array of query objects**. It returns an **array of query results**:</span>

[//]: # 'Example2'

```tsx
function App({ users }) {
  const userQueries = useQueries({
    queries: users.map((user) => {
      return {
        queryKey: ['user', user.id],
        queryFn: () => fetchUserById(user.id),
      }
    }),
  })
}
```

[//]: # 'Example2'

## 요약

- 병렬 쿼리는 여러 개의 useQuery/useInfiniteQuery 훅을 나란히 사용하면 별도의 추가 작업 없이 동시에 실행할 수 있습니다.
- 쿼리 개수가 동적으로 변한다면 useQueries 훅을 활용해 쿼리 객체 배열을 동적으로 생성하여 병렬로 실행할 수 있습니다.
- suspense 모드에서는 기본 병렬 패턴이 동작하지 않으므로 useSuspenseQueries 훅이나 별도 컴포넌트 분리로 병렬 처리를 orchestrate해야 합니다.
- useQueries는 쿼리 결과 배열을 반환하며, 각 쿼리의 상태와 데이터를 개별적으로 관리할 수 있습니다.
- 병렬 쿼리 패턴은 대량의 데이터 패칭, 사용자별/리소스별 동시 요청 등 다양한 실전 상황에서 유용하게 활용됩니다.
