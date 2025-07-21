---
id: overview
title: Overview
---

<span title="TanStack Query(이전 명칭: React Query)는 종종 웹 애플리케이션을 위한 데이터 패칭 라이브러리의 빈자리를 채워주는 도구로 설명됩니다. 좀 더 기술적으로 말하면, 웹 애플리케이션에서 서버 상태의 패칭, 캐싱, 동기화, 업데이트를 매우 쉽게 만들어줍니다.">TanStack Query (formerly known as React Query) is often described as the missing data-fetching library for web applications, but in more technical terms, it makes **fetching, caching, synchronizing and updating server state** in your web applications a breeze.</span>

## <span title="동기">Motivation</span>

<span title="대부분의 핵심 웹 프레임워크는 데이터 패칭이나 업데이트에 대해 일관된 방식을 제공하지 않습니다. 그래서 개발자들은 데이터 패칭에 대해 엄격한 의견을 가진 메타 프레임워크를 만들거나, 각자만의 데이터 패칭 방식을 고안하게 됩니다. 이는 보통 컴포넌트 기반 상태와 부수효과를 조합하거나, 비동기 데이터를 앱 전체에 제공하기 위해 범용 상태 관리 라이브러리를 사용하는 것으로 이어집니다.">Most core web frameworks **do not** come with an opinionated way of fetching or updating data in a holistic way. Because of this developers end up building either meta-frameworks which encapsulate strict opinions about data-fetching, or they invent their own ways of fetching data. This usually means cobbling together component-based state and side-effects, or using more general purpose state management libraries to store and provide asynchronous data throughout their apps.</span>

<span title="대부분의 전통적인 상태 관리 라이브러리는 클라이언트 상태 관리에는 훌륭하지만, 비동기나 서버 상태 관리에는 그다지 적합하지 않습니다. 서버 상태는 완전히 다르기 때문입니다. 예를 들어, 서버 상태는:">While most traditional state management libraries are great for working with client state, they are **not so great at working with async or server state**. This is because **server state is totally different**. For starters, server state:</span>

- <span title="내가 소유하거나 제어하지 않을 수도 있는 원격 위치에 저장됨">Is persisted remotely in a location you may not control or own</span>
- <span title="비동기 API로 패칭/업데이트 필요">Requires asynchronous APIs for fetching and updating</span>
- <span title="공유 소유권을 가지며, 다른 사람이 내 모르게 변경할 수 있음">Implies shared ownership and can be changed by other people without your knowledge</span>
- <span title="주의하지 않으면 앱에서 '오래된' 상태가 될 수 있음">Can potentially become "out of date" in your applications if you're not careful</span>

<span title="앱에서 서버 상태의 본질을 이해하게 되면, 더 많은 도전과제가 생깁니다. 예를 들어:">Once you grasp the nature of server state in your application, **even more challenges will arise** as you go, for example:</span>

- <span title="캐싱... (프로그래밍에서 가장 어려운 일 중 하나)">Caching... (possibly the hardest thing to do in programming)</span>
- <span title="동일 데이터에 대한 여러 요청을 하나로 합치기">Deduping multiple requests for the same data into a single request</span>
- <span title="오래된 데이터를 백그라운드에서 업데이트">Updating "out of date" data in the background</span>
- <span title="데이터가 오래됐는지 판단하기">Knowing when data is "out of date"</span>
- <span title="데이터 업데이트를 최대한 빠르게 반영">Reflecting updates to data as quickly as possible</span>
- <span title="페이지네이션, 지연 로딩 등 성능 최적화">Performance optimizations like pagination and lazy loading data</span>
- <span title="서버 상태의 메모리 관리 및 가비지 컬렉션">Managing memory and garbage collection of server state</span>
- <span title="구조적 공유로 쿼리 결과 메모이제이션">Memoizing query results with structural sharing</span>

<span title="위 목록에 압도당하지 않았다면, 이미 모든 서버 상태 문제를 해결한 것이고 상을 받아야 할 것입니다. 하지만 대부분의 사람들은 아직 이 도전과제들을 모두 해결하지 못했거나, 이제 막 시작한 단계일 것입니다!">If you're not overwhelmed by that list, then that must mean that you've probably solved all of your server state problems already and deserve an award. However, if you are like a vast majority of people, you either have yet to tackle all or most of these challenges and we're only scratching the surface!</span>

<span title="TanStack Query는 서버 상태 관리를 위한 최고의 라이브러리 중 하나입니다. 기본 설정만으로도 매우 잘 동작하며, 앱이 커질수록 원하는 대로 커스터마이즈할 수 있습니다.">TanStack Query is hands down one of the _best_ libraries for managing server state. It works amazingly well **out-of-the-box, with zero-config, and can be customized** to your liking as your application grows.</span>

<span title="TanStack Query를 사용하면 서버 상태의 까다로운 문제와 장애물을 극복하고, 앱 데이터가 나를 통제하기 전에 내가 데이터를 통제할 수 있습니다.">TanStack Query allows you to defeat and overcome the tricky challenges and hurdles of _server state_ and control your app data before it starts to control you.</span>

<span title="좀 더 기술적으로 보면, TanStack Query는 다음과 같은 효과가 있습니다:">On a more technical note, TanStack Query will likely:</span>

- <span title="복잡하고 이해하기 어려운 코드를 많이 제거하고, TanStack Query 로직 몇 줄로 대체할 수 있게 도와줍니다.">Help you remove **many** lines of complicated and misunderstood code from your application and replace with just a handful of lines of TanStack Query logic</span>
- <span title="새 서버 상태 데이터 소스를 연결할 때마다 걱정하지 않고, 앱을 더 유지보수하기 쉽고 새로운 기능을 쉽게 추가할 수 있게 만듭니다.">Make your application more maintainable and easier to build new features without worrying about wiring up new server state data sources</span>
- <span title="앱이 그 어느 때보다 빠르고 반응성 있게 느껴지도록 하여, 최종 사용자에게 직접적인 영향을 줍니다.">Have a direct impact on your end-users by making your application feel faster and more responsive than ever before</span>
- <span title="대역폭을 절약하고 메모리 성능을 높이는 데도 도움이 될 수 있습니다.">Potentially help you save on bandwidth and increase memory performance</span>

[//]: # 'Example'

## <span title="이제 코드로 보여줘!">Enough talk, show me some code already!</span>

<span title="아래 예시에서는 TanStack Query가 가장 기본적이고 단순한 형태로, TanStack Query GitHub 프로젝트의 GitHub 통계를 패칭하는 데 사용되는 모습을 볼 수 있습니다.">In the example below, you can see TanStack Query in its most basic and simple form being used to fetch the GitHub stats for the TanStack Query GitHub project itself:</span>

[Open in StackBlitz](https://stackblitz.com/github/TanStack/query/tree/main/examples/react/simple)

```tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from '@tanstack/react-query'

const queryClient = new QueryClient()

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}

function Example() {
  const { isPending, error, data } = useQuery({
    queryKey: ['repoData'],
    queryFn: () =>
      fetch('https://api.github.com/repos/TanStack/query').then((res) =>
        res.json(),
      ),
  })

  if (isPending) return 'Loading...'

  if (error) return 'An error has occurred: ' + error.message

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👀 {data.subscribers_count}</strong>{' '}
      <strong>✨ {data.stargazers_count}</strong>{' '}
      <strong>🍴 {data.forks_count}</strong>
    </div>
  )
}
```

[//]: # 'Example'
[//]: # 'Materials'

## <span title="이제 설득됐으니, 다음은?">You talked me into it, so what now?</span>

- <span title="공식 TanStack Query 강좌를 수강해보세요(팀 단위 구매도 가능!)">Consider taking the official [TanStack Query Course](https://query.gg?s=tanstack) (or buying it for your whole team!)</span>
- <span title="매우 꼼꼼한 Walkthrough Guide와 API Reference로 TanStack Query를 천천히 익혀보세요.">Learn TanStack Query at your own pace with our amazingly thorough [Walkthrough Guide](../installation.md) and [API Reference](../reference/useQuery.md)</span>

[//]: # 'Materials'

## 요약

- TanStack Query는 서버 상태(원격 데이터)의 패칭, 캐싱, 동기화, 업데이트를 쉽고 일관성 있게 관리할 수 있도록 도와줍니다.
- 서버 상태는 클라이언트 상태와 달리 비동기, 외부 소유, 동시성, 최신성, 캐싱 등 다양한 복잡성을 내포합니다.
- 캐싱, 중복 요청 제거, 백그라운드 업데이트, 데이터 최신성 판단, 성능 최적화, 메모리 관리 등 서버 상태 관리의 주요 과제를 효과적으로 해결할 수 있습니다.
- TanStack Query를 사용하면 복잡한 코드와 상태 관리 로직을 크게 줄이고, 유지보수성과 확장성을 높일 수 있습니다.
- 기본 설정만으로도 강력하게 동작하며, 필요에 따라 다양한 옵션과 커스터마이즈가 가능합니다.
