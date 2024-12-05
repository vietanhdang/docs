# Composition API: Lifecycle Hooks {#composition-api-lifecycle-hooks}

:::info Usage Note
Tất cả các API được liệt kê trên trang này phải được gọi một cách đồng bộ trong giai đoạn `setup()` của một component. Xem [Hướng dẫn - Lifecycle Hooks](/guide/essentials/lifecycle) để biết thêm chi tiết.
:::

## onMounted() {#onmounted}

Đăng ký một hàm callback để được gọi sau khi component đã được mount.

- **Type**

  ```ts
  function onMounted(callback: () => void): void
  ```

- **Details**

  Một component được coi là đã mount sau khi:

  - Tất cả các component con đồng bộ của nó đã được mount (không bao gồm các component không đồng bộ hoặc các component trong cây `<Suspense>`).

  - Cây DOM của chính nó đã được tạo và chèn vào container cha. Lưu ý rằng nó chỉ đảm bảo rằng cây DOM của component nằm trong tài liệu nếu container gốc của ứng dụng cũng nằm trong tài liệu.

  Hook này thường được sử dụng để thực hiện các side effect yêu cầu truy cập vào DOM của component đã render, hoặc để giới hạn mã liên quan đến DOM ở phía client trong một [ứng dụng render phía server.](/guide/scaling-up/ssr).

  **Hook này không được gọi khi render phía server..**

- **Example**

  Truy cập một phần tử thông qua template ref:

  ```vue
  <script setup>
  import { ref, onMounted } from 'vue'

  const el = ref()

  onMounted(() => {
    el.value // <div>
  })
  </script>

  <template>
    <div ref="el"></div>
  </template>
  ```

## onUpdated() {#onupdated}

Đăng ký một hàm callback để được gọi sau khi component đã cập nhật cây DOM của nó do một thay đổi trạng thái reactive.

- **Type**

  ```ts
  function onUpdated(callback: () => void): void
  ```

- **Details**

  Hook của component cha sẽ được gọi sau hook của các component con.

  Hook này được gọi sau bất kỳ cập nhật DOM nào của component, có thể do nhiều thay đổi trạng thái khác nhau, bởi vì nhiều thay đổi trạng thái có thể được gộp lại trong một chu kỳ render để tối ưu hiệu suất. Nếu bạn cần truy cập DOM đã cập nhật sau một thay đổi trạng thái cụ thể, hãy sử dụng [nextTick()](/api/general#nexttick) thay thế.

  **THook này không được gọi trong quá trình render phía server.**

  :::warning
  Không được thay đổi trạng thái component trong hook updated - điều này có thể dẫn đến vòng lặp cập nhật vô hạn!
  :::

- **Example**

  Truy cập DOM đã cập nhật:

  ```vue
  <script setup>
  import { ref, onUpdated } from 'vue'

  const count = ref(0)

  onUpdated(() => {
    // text content should be the same as current `count.value`
    console.log(document.getElementById('count').textContent)
  })
  </script>

  <template>
    <button id="count" @click="count++">{{ count }}</button>
  </template>
  ```

## onUnmounted() {#onunmounted}

Đăng ký một hàm callback để được gọi sau khi component đã bị unmounted.

- **Type**

  ```ts
  function onUnmounted(callback: () => void): void
  ```

- **Details**

  Một component được coi là đã unmounted khi:

  - Tất cả các component con của nó đã bị unmounted.

  - Tất cả các reactive effect liên quan (render effect và computed/watchers được tạo trong `setup()`) đã bị dừng.

  Sử dụng hook này để dọn sạch các side effect được tạo thủ công như timer, DOM event listeners hoặc kết nối server.

  **Hook này không được gọi trong quá trình render phía server.**

- **Example**

  ```vue
  <script setup>
  import { onMounted, onUnmounted } from 'vue'

  let intervalId
  onMounted(() => {
    intervalId = setInterval(() => {
      // ...
    })
  })

  onUnmounted(() => clearInterval(intervalId))
  </script>
  ```

## onBeforeMount() {#onbeforemount}

Đăng ký một hook để được gọi ngay trước khi component sắp được mounting.

- **Type**

  ```ts
  function onBeforeMount(callback: () => void): void
  ```

- **Details**

  Khi hook này được gọi, component đã hoàn tất thiết lập trạng thái reactive, nhưng chưa có nút DOM nào được tạo. Nó sắp thực thi render effect DOM của mình lần đầu tiên.

  **Hook này không được gọi trong quá trình render phía server.**

## onBeforeUpdate() {#onbeforeupdate}

Đăng ký một hook để được gọi ngay trước khi component sắp cập nhật cây DOM của nó do một thay đổi trạng thái reactive.

- **Type**

  ```ts
  function onBeforeUpdate(callback: () => void): void
  ```

- **Details**

  Hook này có thể được sử dụng để truy cập trạng thái DOM trước khi Vue cập nhật DOM. Việc sửa đổi trạng thái component bên trong hook này cũng là an toàn.

  **Hook này không được gọi trong quá trình render phía server.**

## onBeforeUnmount() {#onbeforeunmount}

Registers a hook to be called right before a component instance is to be unmounted.

- **Type**

  ```ts
  function onBeforeUnmount(callback: () => void): void
  ```

- **Details**

  Khi hook này được gọi, instance component vẫn hoàn toàn chức năng.

  **Hook này không được gọi trong quá trình render phía server.**

## onErrorCaptured() {#onerrorcaptured}

Đăng ký một hook để được gọi khi một lỗi lan truyền từ một component con đã được bắt.

- **Type**

  ```ts
  function onErrorCaptured(callback: ErrorCapturedHook): void

  type ErrorCapturedHook = (
    err: unknown,
    instance: ComponentPublicInstance | null,
    info: string
  ) => boolean | void
  ```

- **Details**

  Lỗi có thể được bắt từ các nguồn sau:

  - Render component
  - Trình xử lý sự kiện
  - Lifecycle hooks
  - Hàm `setup()`
  - Watchers
  - Hooks của directive tùy chỉnh
  - Hooks của Transition

  Hook nhận ba đối số: lỗi, instance component đã kích hoạt lỗi, và một chuỗi thông tin chỉ định loại nguồn lỗi.

  :::tip
  Trong môi trường sản xuất, đối số thứ 3 (`info`) sẽ là một mã rút ngắn thay vì chuỗi thông tin đầy đủ. Bạn có thể tìm thấy bảng ánh xạ từ mã đến chuỗi trong [Tham chiếu Mã Lỗi Sản Xuất.](/error-reference/#runtime-errors).
  :::

  Bạn có thể sửa đổi trạng thái trong `errorCaptured()` để hiển thị trạng thái lỗi cho người dùng. Tuy nhiên, điều quan trọng là trạng thái lỗi không được render lại nội dung gốc đã gây ra lỗi; nếu không, component sẽ bị ném vào một vòng render lỗi vô hạn.

  Hook có thể trả về `false` to stop the error from propagating further. See error propagation details below.

  **Error Propagation Rules**

  - Theo mặc định, tất cả các lỗi vẫn được gửi đến [`app.config.errorHandler`](/api/application#app-config-errorhandler) ở cấp ứng dụng nếu nó được định nghĩa, để các lỗi này có thể được báo cáo đến một dịch vụ phân tích tại một nơi duy nhất.

  - Nếu nhiều hook `errorCaptured` tồn tại trên chuỗi kế thừa của component hoặc chuỗi cha, tất cả chúng sẽ được gọi trên cùng một lỗi, theo thứ tự từ dưới lên trên. Điều này tương tự như cơ chế nổi bọt của các sự kiện DOM gốc.

  - Nếu hook `errorCaptured` tự nó ném ra một lỗi, cả lỗi này và lỗi được bắt ban đầu sẽ được gửi đến `app.config.errorHandler`.

  - Hook `errorCaptured` có thể trả về `false` to prevent để ngăn lỗi lan truyền (propagating) xa hơn. Về cơ bản, đây là cách nói "lỗi này đã được xử lý và nên bỏ qua." Nó sẽ ngăn bất kỳ hook `errorCaptured` bổ sung hoặc `app.config.errorHandler` nào khác được gọi cho lỗi này.

## onRenderTracked() <sup class="vt-badge dev-only" /> {#onrendertracked}

Đăng ký một hook debug để được gọi khi một phụ thuộc reactive đã được theo dõi bởi render effect của component.

**Hook này chỉ dành cho chế độ phát triển và không được gọi trong quá trình render phía server.**

- **Type**

  ```ts
  function onRenderTracked(callback: DebuggerHook): void

  type DebuggerHook = (e: DebuggerEvent) => void

  type DebuggerEvent = {
    effect: ReactiveEffect
    target: object
    type: TrackOpTypes /* 'get' | 'has' | 'iterate' */
    key: any
  }
  ```

- **Xem thêm** [Reactivity in Depth](/guide/extras/reactivity-in-depth)

## onRenderTriggered() <sup class="vt-badge dev-only" /> {#onrendertriggered}

Đăng ký một hook debug để được gọi khi một phụ thuộc reactive kích hoạt việc render effect của component được chạy lại.

**Hook này chỉ dành cho chế độ phát triển và không được gọi trong quá trình render phía server.**

- **Type**

  ```ts
  function onRenderTriggered(callback: DebuggerHook): void

  type DebuggerHook = (e: DebuggerEvent) => void

  type DebuggerEvent = {
    effect: ReactiveEffect
    target: object
    type: TriggerOpTypes /* 'set' | 'add' | 'delete' | 'clear' */
    key: any
    newValue?: any
    oldValue?: any
    oldTarget?: Map<any, any> | Set<any>
  }
  ```

- **Xem thêm** [Reactivity in Depth](/guide/extras/reactivity-in-depth)

## onActivated() {#onactivated}

Đăng ký một callback để được gọi sau khi instance component được chèn vào DOM như một phần của một cây được lưu trữ bởi [`<KeepAlive>`](/api/built-in-components#keepalive).

**Hook này không được gọi trong quá trình render phía server.**

- **Type**

  ```ts
  function onActivated(callback: () => void): void
  ```

- **Xem thêm** [Guide - Lifecycle of Cached Instance](/guide/built-ins/keep-alive#lifecycle-of-cached-instance)

## onDeactivated() {#ondeactivated}

Đăng ký một callback để được gọi sau khi instance component bị xóa khỏi DOM như một phần của một cây được lưu trữ bởi [`<KeepAlive>`](/api/built-in-components#keepalive).

**Hook này không được gọi trong quá trình render phía server.**

- **Type**

  ```ts
  function onDeactivated(callback: () => void): void
  ```

- **Xem thêm** [Guide - Lifecycle of Cached Instance](/guide/built-ins/keep-alive#lifecycle-of-cached-instance)

## onServerPrefetch() <sup class="vt-badge" data-text="SSR only" /> {#onserverprefetch}

Đăng ký một hàm không đồng bộ để được giải quyết trước khi instance component được render trên server.

- **Type**

  ```ts
  function onServerPrefetch(callback: () => Promise<any>): void
  ```

- **Details**

  Nếu callback trả về một Promise, trình render phía server sẽ đợi cho đến khi Promise được giải quyết trước khi render component.

  Hook này chỉ được gọi trong quá trình render phía server và có thể được sử dụng để thực hiện việc tìm nạp dữ liệu chỉ dành cho server.

- **Example**

  ```vue
  <script setup>
  import { ref, onServerPrefetch, onMounted } from 'vue'

  const data = ref(null)

  onServerPrefetch(async () => {
    // component is rendered as part of the initial request
    // pre-fetch data on server as it is faster than on the client
    data.value = await fetchOnServer(/* ... */)
  })

  onMounted(async () => {
    if (!data.value) {
      // if data is null on mount, it means the component
      // is dynamically rendered on the client. Perform a
      // client-side fetch instead.
      data.value = await fetchOnClient(/* ... */)
    }
  })
  </script>
  ```

- **Xem thêm** [Server-Side Rendering](/guide/scaling-up/ssr)
