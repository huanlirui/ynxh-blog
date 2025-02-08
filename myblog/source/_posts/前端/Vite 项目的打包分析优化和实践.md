# Vite 项目的打包分析优化和实践

## 1. 问题背景

在 Vite 项目中，默认的打包配置可能不够优化，特别是对于需要嵌入到 WebView 或 iframe 中的 H5 项目，需要针对性的优化以提升加载性能。

## 2. 优化方案

### 2.1 安装必要的依赖

```bash
# 打包分析工具
pnpm add -D rollup-plugin-visualizer
# Gzip 压缩插件
pnpm add -D vite-plugin-compression
```

### 2.2 完整的 vite.config.ts 配置

```typescript
import { defineConfig, type UserConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { VantResolver } from '@vant/auto-import-resolver'
import { visualizer } from 'rollup-plugin-visualizer'
import viteCompression from 'vite-plugin-compression'

// https://vite.dev/config/
export default defineConfig(async (): Promise<UserConfig> => {
  const UnoCSS = (await import('unocss/vite')).default

  return {
    plugins: [
      vue(),
      UnoCSS(),
      AutoImport({
        resolvers: [VantResolver()],
        dts: true,
      }),
      Components({
        resolvers: [VantResolver({ importStyle: true })],
        dts: true,
      }),
      visualizer({
        open: true,
        gzipSize: true,
        brotliSize: true,
        filename: '.vite/stats.html', // 分析报告生成位置
      }),
      // gzip 压缩
      viteCompression({
        verbose: true,
        disable: false,
        threshold: 10240, // 10kb 以上的文件才会被压缩
        algorithm: 'gzip',
        ext: '.gz',
        deleteOriginFile: false, // 是否删除原文件
      }),
    ],
    resolve: {
      alias: {
        '@': '/src',
      },
    },
    build: {
      chunkSizeWarningLimit: 600,
      minify: 'esbuild',
      cssCodeSplit: true,
      sourcemap: false,
      target: ['es2015', 'chrome87', 'safari13'],
      reportCompressedSize: false,
      rollupOptions: {
        output: {
          manualChunks: {
            // 核心依赖
            'vue-vendor': ['vue', 'vue-router', 'pinia'],
            // UI 框架
            vant: ['vant'],
            // 图表库
            echarts: ['echarts'],
            // 工具库
            utils: ['lodash-es', 'alova', 'sm-crypto', 'qs'],
          },
          chunkFileNames: 'assets/js/[name]-[hash].js',
          entryFileNames: 'assets/js/[name]-[hash].js',
          assetFileNames: (assetInfo) => {
            if (!assetInfo.name) return 'assets/[ext]/[name]-[hash][extname]'

            if (/.css$/.test(assetInfo.name)) return 'assets/css/[name]-[hash][extname]'
            if (/.(png|jpe?g|gif|svg|webp)$/.test(assetInfo.name))
              return 'assets/img/[name]-[hash][extname]'
            if (/.(woff2?|eot|ttf|otf)$/.test(assetInfo.name))
              return 'assets/fonts/[name]-[hash][extname]'

            return 'assets/[ext]/[name]-[hash][extname]'
          },
        },
      },
    },
  }
})
```

## 3. 优化点说明

### 3.1 构建优化

- 使用 `esbuild` 作为压缩工具，相比 terser 速度更快
- 开启 `cssCodeSplit`，实现 CSS 的按需加载
- 设置合适的浏览器目标，优化输出代码
- 关闭 sourcemap，减小生产环境代码体积

### 3.2 分包策略

- 核心框架单独打包（vue + router + pinia）
- UI 框架独立（vant）
- 工具库集中管理
- 图表库单独打包
  这样的分包策略可以：

1. 更好地利用浏览器缓存
2. 避免重复加载
3. 实现按需加载

### 3.3 资源管理

- 静态资源按类型分类（css/img/fonts）
- 使用 hash 确保缓存更新
- 合理的文件命名策略

### 3.4 压缩优化

- 使用 gzip 压缩大文件（>10KB）
- 保留原始文件以兼容不支持 gzip 的环境

## 4. 服务器配置

如果使用 Nginx 服务器，需要添加以下配置以支持 gzip：

```nginx
gzip on;
gzip_static on;  # 开启静态 gzip 功能
gzip_min_length 10k;  # 小于 10k 的文件不压缩
gzip_comp_level 6;  # 压缩级别 1-9，越大压缩率越高，但越耗 CPU
```

## 5. 为什么不需要某些优化

### 5.1 PWA

- 在 WebView/iframe 环境中，PWA 特性基本无法使用
- 离线访问、添加到主屏幕等功能在嵌入式场景下没有意义

### 5.2 预加载优化

- 在 WebView/iframe 环境中，预加载可能会被宿主环境的策略影响
- 可能会浪费资源，因为用户可能并不会访问所有预加载的内容

### 5.3 资源预加载

- WebView/iframe 环境中，DNS 预解析等优化效果有限
- 字体预加载等可以通过其他方式处理

## 6. 如何验证优化效果

1. 运行构建命令：

```bash
npm run build
```

2. 查看分析报告：

- 构建完成后会自动打开 `.vite/stats.html`
- 可以看到各个模块的大小分布
- 检查分包是否合理
- 观察压缩效果

3. 实际部署测试：

- 检查首屏加载时间
- 监控资源加载顺序
- 验证缓存效果

## 7. 注意事项

1. 分包策略要根据实际项目依赖调整
2. 压缩阈值可以根据项目实际情况调整
3. 浏览器目标版本要根据实际用户群体来设置
4. 定期检查和更新依赖，保持最佳性能
