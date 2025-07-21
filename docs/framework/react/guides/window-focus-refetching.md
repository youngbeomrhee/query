---
id: window-focus-refetching
title: Window Focus Refetching
---

<span title="사용자가 앱을 떠났다가 돌아왔을 때 쿼리 데이터가 stale 상태라면, TanStack Query는 자동으로 백그라운드에서 신선한 데이터를 요청합니다. 이 동작은 refetchOnWindowFocus 옵션으로 전역 또는 쿼리별로 비활성화할 수 있습니다.">If a user leaves your application and returns and the query data is stale, **TanStack Query automatically requests fresh data for you in the background**. You can disable this globally or per-query using the `refetchOnWindowFocus` option:</span>

#### <span title="전역 비활성화">Disabling Globally</span>

[//]: # 'Example'

```tsx
//
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false, // default: true
    },
  },
})

function App() {
  return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
}
```

[//]: # 'Example'

#### <span title="쿼리별 비활성화">Disabling Per-Query</span>

[//]: # 'Example2'

```tsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  refetchOnWindowFocus: false,
})
```

[//]: # 'Example2'

## <span title="커스텀 윈도우 포커스 이벤트">Custom Window Focus Event</span>

<span title="특이한 경우, 윈도우 포커스 이벤트를 직접 관리해 TanStack Query의 revalidate를 트리거하고 싶을 수 있습니다. 이를 위해 focusManager.setEventListener 함수를 제공하며, 이 함수는 윈도우가 포커스될 때 실행할 콜백을 받아 직접 이벤트를 설정할 수 있습니다. setEventListener를 호출하면 기존 핸들러(대부분 기본 핸들러)가 제거되고, 새 핸들러가 사용됩니다. 아래는 기본 핸들러 예시입니다:">In rare circumstances, you may want to manage your own window focus events that trigger TanStack Query to revalidate. To do this, TanStack Query provides a `focusManager.setEventListener` function that supplies you the callback that should be fired when the window is focused and allows you to set up your own events. When calling `focusManager.setEventListener`, the previously set handler is removed (which in most cases will be the default handler) and your new handler is used instead. For example, this is the default handler:</span>

[//]: # 'Example3'

```tsx
focusManager.setEventListener((handleFocus) => {
  // Listen to visibilitychange
  if (typeof window !== 'undefined' && window.addEventListener) {
    const visibilitychangeHandler = () => {
      handleFocus(document.visibilityState === 'visible')
    }
    window.addEventListener('visibilitychange', visibilitychangeHandler, false)
    return () => {
      // Be sure to unsubscribe if a new handler is set
      window.removeEventListener('visibilitychange', visibilitychangeHandler)
    }
  }
})
```

[//]: # 'Example3'
[//]: # 'ReactNative'

## <span title="React Native에서 포커스 관리">Managing Focus in React Native</span>

<span title="window의 이벤트 리스너 대신, React Native는 AppState 모듈을 통해 포커스 정보를 제공합니다. AppState의 'change' 이벤트를 사용해 앱 상태가 'active'로 바뀔 때 업데이트를 트리거할 수 있습니다.">Instead of event listeners on `window`, React Native provides focus information through the [`AppState` module](https://reactnative.dev/docs/appstate#app-states). You can use the `AppState` "change" event to trigger an update when the app state changes to "active":</span>

```tsx
import { AppState } from 'react-native'
import { focusManager } from '@tanstack/react-query'

function onAppStateChange(status: AppStateStatus) {
  if (Platform.OS !== 'web') {
    focusManager.setFocused(status === 'active')
  }
}

useEffect(() => {
  const subscription = AppState.addEventListener('change', onAppStateChange)

  return () => subscription.remove()
}, [])
```

[//]: # 'ReactNative'

## <span title="포커스 상태 관리">Managing focus state</span>

[//]: # 'Example4'

```tsx
import { focusManager } from '@tanstack/react-query'

// Override the default focus state
focusManager.setFocused(true)

// Fallback to the default focus check
focusManager.setFocused(undefined)
```

[//]: # 'Example4'

## 요약

- 사용자가 앱을 떠났다가 돌아오면, 쿼리 데이터가 stale 상태일 때 TanStack Query가 자동으로 백그라운드 refetch를 수행합니다.
- refetchOnWindowFocus 옵션을 통해 전역 또는 쿼리별로 이 동작을 비활성화할 수 있습니다.
- focusManager.setEventListener를 활용하면 윈도우 포커스 이벤트를 직접 커스터마이즈하여 쿼리의 revalidate 타이밍을 제어할 수 있습니다.
- React Native 환경에서는 AppState 모듈을 활용해 포커스 상태를 감지하고, focusManager.setFocused로 쿼리 refetch를 트리거할 수 있습니다.
- 포커스 상태를 수동으로 오버라이드하거나, 기본 체크로 되돌릴 수도 있어 다양한 환경에서 유연하게 활용 가능합니다.
