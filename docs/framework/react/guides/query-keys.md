---
id: query-keys
title: Query Keys
---

<span title="TanStack Query는 쿼리 키를 기반으로 쿼리 캐싱을 관리합니다. 쿼리 키는 최상위에서 배열이어야 하며, 단일 문자열 배열처럼 단순할 수도 있고, 여러 문자열과 중첩 객체로 복잡할 수도 있습니다. 쿼리 키가 JSON.stringify로 직렬화 가능하고, 쿼리 데이터에 대해 고유하다면 사용할 수 있습니다!">At its core, TanStack Query manages query caching for you based on query keys. Query keys have to be an Array at the top level, and can be as simple as an Array with a single string, or as complex as an array of many strings and nested objects. As long as the query key is serializable using `JSON.stringify`, and **unique to the query's data**, you can use it!</span>

## <span title="단순 쿼리 키">Simple Query Keys</span>

<span title="키의 가장 단순한 형태는 상수 값 배열입니다. 이 포맷은 다음에 유용합니다:">The simplest form of a key is an array with constants values. This format is useful for:</span>

- <span title="일반적인 리스트/인덱스 리소스">Generic List/Index resources</span>
- <span title="비계층적 리소스">Non-hierarchical resources</span>

[//]: # 'Example'

```tsx
// A list of todos
useQuery({ queryKey: ['todos'], ... })

// Something else, whatever!
useQuery({ queryKey: ['something', 'special'], ... })
```

[//]: # 'Example'

## <span title="변수가 포함된 배열 키">Array Keys with variables</span>

<span title="쿼리가 데이터를 고유하게 설명하기 위해 더 많은 정보가 필요할 때, 문자열과 여러 직렬화 가능한 객체를 포함하는 배열을 사용할 수 있습니다. 이 방식은 다음에 유용합니다:">When a query needs more information to uniquely describe its data, you can use an array with a string and any number of serializable objects to describe it. This is useful for:</span>

- <span title="계층적 또는 중첩 리소스">Hierarchical or nested resources</span>
  - <span title="아이디, 인덱스, 기타 원시값을 전달해 아이템을 고유하게 식별하는 것이 일반적입니다.">It's common to pass an ID, index, or other primitive to uniquely identify the item</span>
- <span title="추가 파라미터가 있는 쿼리">Queries with additional parameters</span>
  - <span title="추가 옵션 객체를 전달하는 것이 일반적입니다.">It's common to pass an object of additional options</span>

[//]: # 'Example2'

```tsx
// An individual todo
useQuery({ queryKey: ['todo', 5], ... })

// An individual todo in a "preview" format
useQuery({ queryKey: ['todo', 5, { preview: true }], ...})

// A list of todos that are "done"
useQuery({ queryKey: ['todos', { type: 'done' }], ... })
```

[//]: # 'Example2'

## <span title="쿼리 키는 결정적으로 해시됩니다!">Query Keys are hashed deterministically!</span>

<span title="즉, 객체 내 키의 순서와 상관없이 아래 쿼리들은 모두 동일하게 간주됩니다:">This means that no matter the order of keys in objects, all of the following queries are considered equal:</span>

[//]: # 'Example3'

```tsx
useQuery({ queryKey: ['todos', { status, page }], ... })
useQuery({ queryKey: ['todos', { page, status }], ...})
useQuery({ queryKey: ['todos', { page, status, other: undefined }], ... })
```

[//]: # 'Example3'

<span title="하지만 아래 쿼리 키들은 서로 다릅니다. 배열 아이템의 순서가 중요합니다!">The following query keys, however, are not equal. Array item order matters!</span>

[//]: # 'Example4'

```tsx
useQuery({ queryKey: ['todos', status, page], ... })
useQuery({ queryKey: ['todos', page, status], ...})
useQuery({ queryKey: ['todos', undefined, page, status], ...})
```

[//]: # 'Example4'

## <span title="쿼리 함수가 변수를 의존한다면, 쿼리 키에 포함하세요">If your query function depends on a variable, include it in your query key</span>

<span title="쿼리 키는 가져오는 데이터를 고유하게 설명하므로, 쿼리 함수에서 사용하는(변경되는) 모든 변수를 쿼리 키에 포함해야 합니다. 예시:">Since query keys uniquely describe the data they are fetching, they should include any variables you use in your query function that **change**. For example:</span>

[//]: # 'Example5'

```tsx
function Todos({ todoId }) {
  const result = useQuery({
    queryKey: ['todos', todoId],
    queryFn: () => fetchTodoById(todoId),
  })
}
```

[//]: # 'Example5'

<span title="쿼리 키는 쿼리 함수의 의존성 역할도 합니다. 의존 변수를 쿼리 키에 추가하면 쿼리가 독립적으로 캐싱되고, 변수가 바뀔 때마다 쿼리가 자동으로 refetch됩니다(staleTime 설정에 따라). 자세한 내용과 예시는 exhaustive-deps 문서를 참고하세요.">Note that query keys act as dependencies for your query functions. Adding dependent variables to your query key will ensure that queries are cached independently, and that any time a variable changes, _queries will be refetched automatically_ (depending on your `staleTime` settings). See the [exhaustive-deps](../../../../eslint/exhaustive-deps.md) section for more information and examples.</span>

[//]: # 'Materials'

## <span title="추가 자료">Further reading</span>

<span title="대규모 앱에서 쿼리 키를 조직화하는 팁은 Effective React Query Keys와 커뮤니티 리소스의 Query Key Factory Package를 참고하세요.">For tips on organizing Query Keys in larger applications, have a look at [Effective React Query Keys](../../community/tkdodos-blog.md#8-effective-react-query-keys) and check the [Query Key Factory Package](../../community/community-projects.md#query-key-factory) from
the Community Resources.</span>

[//]: # 'Materials'

## 요약

- TanStack Query의 쿼리 키는 배열 형태로, 쿼리 데이터의 고유성과 캐싱을 결정하는 핵심 요소입니다.
- 단순한 문자열 배열부터, 여러 변수와 객체가 포함된 복잡한 배열까지 다양한 형태로 쿼리 키를 정의할 수 있습니다.
- 쿼리 키는 순서와 값이 모두 중요하며, 객체 내 프로퍼티 순서는 상관없지만 배열 내 순서는 다르면 다른 쿼리로 간주됩니다.
- 쿼리 함수가 변수를 의존한다면, 반드시 쿼리 키에 해당 변수를 포함해야 쿼리 캐싱과 refetch가 올바르게 동작합니다.
- 대규모 앱에서는 쿼리 키의 일관성과 관리가 매우 중요하며, 쿼리 키 팩토리 패턴 등으로 체계적으로 관리하는 것이 좋습니다.
