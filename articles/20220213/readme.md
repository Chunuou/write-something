# Vue 3 中可能会带来性能问题

在网上看见了关于 vue3 性能问题的[讨论](https://www.v2ex.com/t/829796)，写了点代码，验证下。

原始数组：

```javascript
const raw = [];

for (let i = 0; i < 100; i++) {
      raw.push({ a: 'a' });
}
```

vue 2:

```javascript
new Vue({
    el: '#app',

    data() {
        return {
            reactive: raw,
        };
    },

    created() {
        const start = new Date().getTime();

        for (let index = 0; index < 10000; index++) {
            this.reactive.forEach((i) => {
                i.a = index;
            });
        }

        const end = new Date().getTime();

        console.log(end - start); // 250
    },
});
```

vue3:

```javascript
Vue.createApp({
    el: '#app',

    data() {
        return {
            reactive: raw,
        };
    },

    created() {
        const start = new Date().getTime();

        for (let index = 0; index < 10000; index++) {
            this.reactive.forEach((i) => {
                i.a = index;
            });
        }

        const end = new Date().getTime();

        console.log(end - start); // 2000
    },
}).mount('#app');
```

vue3(with `toRaw`):

```javascript
Vue.createApp({
    el: '#app',

    data() {
        return {
            reactive: raw,
        };
    },

    created() {
        const start = new Date().getTime();

        const raw = Vue.toRaw(this.reactive)

        for (let index = 0; index < 10000; index++) {
            raw.forEach((i) => {
                i.a = index;
            });
        }

        this.reactive = Vue.reactive(raw)

        const end = new Date().getTime();

        console.log(end - start); // 20
    },
}).mount('#app');
```

结论很明显了——[Proxy API](https://www.cnblogs.com/zmj97/p/10954968.html) 及使用 proxy 的 vue3 可能会”引发“性能问题，需要注意。
