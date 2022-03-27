---
layout: post
title: "Lazy Load Hook: Nhiệm vụ bất khả thi"
date: 2022-03-11 07:00:00 +0700
categories: react, javascript, frontend
---

Lazy Load một cái Hook, không phải là tạo ra một cái Hook để Lazy Load cái gì đó. Các bác bắt đầu thấy gì đó sai sai rồi đúng không. Hình như mâu thuẫn với nguyên tắc của React `No rendered more hooks than during the previous render`? Tuy vậy hãy thử xem chúng ta có thể workaround thế nào nhé

## Cách 1: Load ở Parent Component, và render Child Component (chứa hook) khi thư viện load xong

Đầu tiên chúng ta có `ChildComponent`

```
const ChildComponent = () => {
  return <div>Child</div>
}
```

Tại `ParentComponent`

```
const ParentComponent = () => {
  const [loaded, setLoaded] = useState(false)

  useEffect(() => {
    loadComponent().then(() => setLoaded(true))
  }, [])

  return (
    <div>
      <AnotherComponent>
      {loaded && <AnotherComponentWithHook>}
    </div>
  )
}
```

Như vậy hook sẽ được bọc trong một component mà ở đó nó sẽ chờ cho thư viện được load. Tư tưởng khá đơn giản đúng không? Tuy nhiên làm thế này thì không xứng tầm Vozer lương 2000$ lắm, và phát sinh nhiều code duplicate

## Cách 2: Kệ cho load component với hook bị lỗi và cho rerender lại sau khi hook load xong

Chắc các bác đang nghĩ: Ủa, lỗi xong nó crash luôn thì ai về nhà nấy rồi còn gì? Giờ đến lúc `Suspense` trong React ra tay. Suspense sẽ hoạt động theo chu trình sau:
- Component throws một cái Promise trong lúc Render (thông qua React.lazy)
- Suspense bắt Promise và render Fallback
- Promise được giải phóng (aka Promise resolve)
- Suspense render lại Component

Giờ ta sẽ chế một cái mini Hook để throw Promise, promise có nhiệm vụ yêu cầu load thư viện ta cần. Và để cho quá trình này hiệu quả ta sẽ cache nó trong bộ nhớ chả hạn
```
const cache = {}

export const useSuspense = (importPromise, cacheKey) => {
  const cachedModule = cache[cacheKey]
  if (cachedModule) return cachedModule

  throw importPromise
    .then((mod) => (cache[cacheKey] = mod))
};
```

Có điều, nhỡ đâu nó load lỗi thì sao? Lúc đấy lệnh import sẽ bị loop vô tận, vì vậy nên ta sẽ thêm errorsCache vào để phòng ngừa

```
// useSuspense.ts

const cache = {}
const errorsCache = {}

export const useSuspense = (importPromise, cacheKey) => {
  const cachedModule = cache[cacheKey]
  if (cachedModule) return cachedModule

  if (errorsCache[cacheKey]) throw errorsCache[cacheKey]

  throw importPromise
    .then((mod) => (cache[cacheKey] = mod))
    .catch((err) => {
      errorsCache[cacheKey] = err
    })
};
```

Tại ChildComponent chúng ta sử dụng như sau, và đừng quên bọc trong Suspense
```
const ChildComponent = () => {
  const { useForm } = useSuspense(import("react-hook-form"), "react-hook-form") // VD thư viện này
  const { register, handleSubmit, watch, errors } = useForm()
}

...
// Bọc hẳn bên ngoài, không bọc dưới return của ChildComponent
<Suspense fallback={<div>Loading</div>}> 
  <ChildComponent/>
</Suspense>
```

⚠️ Kĩ thuật này có tính over-engineering. Việc lazy load một hook (thứ gần như rất nhẹ) sẽ khiến thời gian load request khả năng cao lâu hơn là gộp chung trong một chunk & waterfall request (badly). Use case cháu dùng duy nhất là để load những hook mà bị lỗi server-side trên Next.js

