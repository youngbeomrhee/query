## TanStack Query의 Structural Sharing 핵심 개념 및 구현 정리

### 🔍 **핵심 개념**

**Structural Sharing**은 TanStack Query에서 데이터의 참조 안정성을 보장하기 위한 최적화 기법입니다. 새로 받아온 데이터와 기존 데이터를 깊이 비교하여, 실제로 변경된 부분만 새로운 참조로 교체하고 변경되지 않은 부분은 기존 참조를 유지합니다.

### 🏗️ **핵심 구현 구조**

#### 1. **`replaceEqualDeep` 함수** (`packages/query-core/src/utils.ts`)
```typescript
export function replaceEqualDeep<T>(a: unknown, b: T): T {
  if (a === b) {
    return a  // 동일한 참조면 기존 값 반환
  }

  const array = isPlainArray(a) && isPlainArray(b)

  if (array || (isPlainObject(a) && isPlainObject(b))) {
    const aItems = array ? a : Object.keys(a)
    const aSize = aItems.length
    const bItems = array ? b : Object.keys(b)
    const bSize = bItems.length
    const copy: any = array ? [] : {}
    const aItemsSet = new Set(aItems)

    let equalItems = 0

    for (let i = 0; i < bSize; i++) {
      const key = array ? i : bItems[i]
      if (
        ((!array && aItemsSet.has(key)) || array) &&
        a[key] === undefined &&
        b[key] === undefined
      ) {
        copy[key] = undefined
        equalItems++
      } else {
        copy[key] = replaceEqualDeep(a[key], b[key])  // 재귀적 비교
        if (copy[key] === a[key] && a[key] !== undefined) {
          equalItems++
        }
      }
    }

    return aSize === bSize && equalItems === aSize ? a : copy
  }

  return b  // 다른 타입이면 새 값 반환
}
```

#### 2. **`replaceData` 함수** (`packages/query-core/src/utils.ts`)
```typescript
export function replaceData<TData, TOptions extends QueryOptions<any, any, any, any>>(
  prevData: TData | undefined, 
  data: TData, 
  options: TOptions
): TData {
  if (typeof options.structuralSharing === 'function') {
    return options.structuralSharing(prevData, data) as TData
  } else if (options.structuralSharing !== false) {
    if (process.env.NODE_ENV !== 'production') {
      try {
        return replaceEqualDeep(prevData, data)
      } catch (error) {
        console.error(
          `Structural sharing requires data to be JSON serializable. To fix this, turn off structuralSharing or return JSON-serializable data from your queryFn. [${options.queryHash}]: ${error}`,
        )
        throw error
      }
    }
    return replaceEqualDeep(prevData, data)
  }
  return data
}
```

### 🎯 **주요 사용처**

#### 1. **Query 데이터 설정** (`packages/query-core/src/query.ts`)
```typescript
setData(newData: TData, options?: SetDataOptions & { manual: boolean }): TData {
  const data = replaceData(this.state.data, newData, this.options)
  
  this.#dispatch({
    data,
    type: 'success',
    dataUpdatedAt: options?.updatedAt,
    manual: options?.manual,
  })

  return data
}
```

#### 2. **QueryObserver에서 결과 생성** (`packages/query-core/src/queryObserver.ts`)
```typescript
// placeholderData 처리시
data = replaceData(
  prevResult?.data,
  placeholderData as unknown,
  options,
) as TData

// select 함수 결과 처리시
data = replaceData(prevResult?.data, data, options)
```

### 🔧 **유틸리티 함수들**

#### 1. **`isPlainObject`** - 순수 객체 판별
```typescript
export function isPlainObject(o: any): o is Object {
  if (!hasObjectPrototype(o)) {
    return false
  }

  const ctor = o.constructor
  if (ctor === undefined) {
    return true
  }

  const prot = ctor.prototype
  if (!hasObjectPrototype(prot)) {
    return false
  }

  if (!prot.hasOwnProperty('isPrototypeOf')) {
    return false
  }

  return true
}
```

#### 2. **`isPlainArray`** - 순수 배열 판별
```typescript
export function isPlainArray(value: unknown) {
  return Array.isArray(value) && value.length === Object.keys(value).length
}
```

### ⚙️ **옵션 설정**

#### 1. **타입 정의** (`packages/query-core/src/types.ts`)
```typescript
/**
 * Set this to `false` to disable structural sharing between query results.
 * Set this to a function which accepts the old and new data and returns resolved data of the same type to implement custom structural sharing logic.
 * Defaults to `true`.
 */
structuralSharing?:
  | boolean
  | ((oldData: unknown | undefined, newData: unknown) => unknown)
```

#### 2. **설정 방법**
```typescript
// 전역 비활성화
queryClient.setDefaultOptions({
  queries: {
    structuralSharing: false,
  },
})

// 개별 쿼리에서 비활성화
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  structuralSharing: false,
})

// 커스텀 structural sharing 함수
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  structuralSharing: (oldData, newData) => {
    // 커스텀 로직
    return customComparison(oldData, newData) ? oldData : newData
  },
})
```

### 🧪 **동작 원리 예시**

```typescript
const prev = {
  todo: { id: '1', meta: { createdAt: 0 }, state: { done: false } },
  otherTodo: { id: '2', meta: { createdAt: 0 }, state: { done: true } },
}

const next = {
  todo: { id: '1', meta: { createdAt: 0 }, state: { done: true } },
  otherTodo: { id: '2', meta: { createdAt: 0 }, state: { done: true } },
}

const result = replaceEqualDeep(prev, next)

// 결과:
// - result.todo.meta === prev.todo.meta (변경되지 않았으므로 기존 참조 유지)
// - result.otherTodo === prev.otherTodo (완전히 동일하므로 기존 참조 유지)
// - result.todo.state !== prev.todo.state (변경되었으므로 새 참조)
```

### 🚨 **제약사항 및 주의점**

1. **JSON 호환성**: JSON으로 직렬화 가능한 데이터만 지원
2. **순환 참조**: 순환 참조가 있는 객체는 처리할 수 없음
3. **성능**: 매우 큰 객체의 경우 성능 이슈가 있을 수 있음
4. **프레임워크별 차이**: Solid.js에서는 기본적으로 비활성화됨

### 💡 **실전 활용**

1. **React.useEffect 최적화**: 참조가 안정되어 불필요한 effect 실행 방지
2. **React.memo 최적화**: props 비교 시 참조 안정성으로 리렌더링 방지
3. **성능 최적화**: 대용량 데이터에서 변경된 부분만 새로 생성하여 메모리 효율성 향상

이 구조를 통해 TanStack Query는 데이터의 불변성을 유지하면서도 참조 안정성을 보장하여, React 등의 프레임워크에서 최적의 성능을 제공합니다.
