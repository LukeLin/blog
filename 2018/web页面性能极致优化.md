## 详细讲解几个优化点：
减少请求：
合并文件、css精灵图、小图base64、

缓存：
添加Expires、Cache-Control或者ETag响应头、service worker

减少体积：
文件压缩、gzip压缩、减少cookie体积、代码tree shaking、图片优化

减少链路：
减少DNS查询、避免重定向、CDN

预先操作：
预拉取资源、DNS prefetch

懒操作：
非必须组件延迟加载、

优先渲染：
Flush Buffer Early、服务端渲染、预渲染
