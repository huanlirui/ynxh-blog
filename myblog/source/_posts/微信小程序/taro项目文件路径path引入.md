config/index.js 增加如下配置

  alias: {
    '@/api': path.resolve(__dirname, '..', 'src/api'),
    '@/assets': path.resolve(__dirname, '..', 'src/assets'),
    '@/config': path.resolve(__dirname, '..', 'src/config'),
    '@/models': path.resolve(__dirname, '..', 'src/models'),
    '@/components': path.resolve(__dirname, '..', 'src/components'),
    '@/utils': path.resolve(__dirname, '..', 'src/utils'),
    '@/package': path.resolve(__dirname, '..', 'package.json'),
    '@/project': path.resolve(__dirname, '..', 'project.config.json')
  },