# 使用 sass 替换 node-sass

`@vue/cli-service` 的版本号为 `4.2.2`，可以直接使用 `sass` 替换，无需修改配置。

1. 删除 `"node-sass": "^4.14.1"`
1. 添加 `"sass": "1.49.9"`
1. `/deep/` 更改为 `::v-deep`