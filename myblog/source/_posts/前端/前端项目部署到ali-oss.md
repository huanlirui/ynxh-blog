---
cover: /img/javascript.jpg
---

# 阿里云对象存储(OSS)部署前端项目并使用自动脚本上传

## **前言**

使用阿里云 OSS + CDN 方案的几大好处：

- 成本低廉。OSS+CDN 部署自己的网站每个月的花费远比自己买 ECS 服务器部署网站花费要少得多
- 大幅降低运维成本。直接使用现成的云服务了，无需花精力去维护 ECS 了。
- 极高的可用性。无论阿里云的 OSS 还是 CDN，都已经做好了高可用性，几乎可以保证网站始终可访问
- 极高的访问速度。ECS 带宽毕竟有限的，高带宽意味着超高的费用。但是用 OSS+CDN 这种天然分布式的架构部署网站很轻松的解决了带宽问题，极大地提升了用户的访问体验。
- OSS 会自动帮你压缩，使得你的页面加载速度极快。

## **配置 阿里云对象存储(OSS)**

首先登陆阿里云 oss 控制台，点击创建一个 bucket。

![img](https://e8ihplx6a2.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDQyYWU3MzM1ZmU4YmIzN2RjNmMwYjQ5OTgxZDFmOGVfOVZCeW9ka1dCcDVwQzR2RlFvaW94VVdaSTVIeThYNkxfVG9rZW46Ym94Y25VUVpjb0pIcjFSN0FCejRLVnN5ZzJmXzE2Nzc1NjgzNjk6MTY3NzU3MTk2OV9WNA)

根据自己的需求选择参数。

然后就创建了一个 bucket。

![img](https://e8ihplx6a2.feishu.cn/space/api/box/stream/download/asynccode/?code=NThhYWQzNDYxNmNiNTZkZjllNWM1OGM3MWFkN2ZiMGFfb1VlUUdjYkdnTzladUwxd3pHbGsxVnU2WmRIbVhXd3VfVG9rZW46Ym94Y25yMlNIOUVjZEF2bzVGcVNCSGR5SUZlXzE2Nzc1NjgzNjk6MTY3NzU3MTk2OV9WNA)

为 bucket 配置域名，在上图中，将阿里云外网访问 Bucket 域名记录下来。然后在 DNS 控制台添加记录解析。然后回到 oss 控制台，在**域名管理**选项 将你刚刚 DNS 解析的域名 绑定上去。这样就可以通过自己设置的二级域名访问到自己的项目了。

![img](https://e8ihplx6a2.feishu.cn/space/api/box/stream/download/asynccode/?code=NjE0ZTIyNjE4MWUwMzQ3NzM4YmRkMjJjNGE0NTYyMmNfSXNFcVRFcHRCdUJSekszemdKQ283ZW5PYWtBb2JlOVdfVG9rZW46Ym94Y25zVzJtZlc4TkI5WWlxUGFlMkNPQ2tiXzE2Nzc1NjgzNjk6MTY3NzU3MTk2OV9WNA)

![img](https://e8ihplx6a2.feishu.cn/space/api/box/stream/download/asynccode/?code=YWRjNDlkN2UyNGJkOTJhZGJmMmQyYmExMDBmYTQzOTBfcEZmallCUDBGNXlWN1c3eXRxQ3B5MDVsZ1VFSmlZTWZfVG9rZW46Ym94Y240M3NRNWZhUk5zQm9GYVF5NHpDZ1FkXzE2Nzc1NjgzNjk6MTY3NzU3MTk2OV9WNA)

在**基础设置**找到静态页面 设置默认首页的文件名 一般都是 index.html，如果有 404 页面也可以配置。

![img](https://e8ihplx6a2.feishu.cn/space/api/box/stream/download/asynccode/?code=OWQzNGZiMTIyYTIwYTk2MTk5NWJhZDM0MDc4ZmRmOTRfYmpuSkVxb2s1VnVPNnk5aTd4MXlIRXRnaXN1aEJnWmRfVG9rZW46Ym94Y242SUhKaEVBWWM3UHVCTHBJVjdNR09kXzE2Nzc1NjgzNjk6MTY3NzU3MTk2OV9WNA)

接下来就只要将自己的打包出来的静态文件 通过 deploy.js 脚本上传到 OSS 上就行了

## **配置 deploy.js**

```JavaScript
const fs = require("fs");
const path = require("path");
const util = require("util");
const OSS = require("ali-oss");

const promisifyReaddir = util.promisify(fs.readdir);
const promisifyStat = util.promisify(fs.stat);

const client = new OSS({
  // yourregion填写Bucket所在地域。以华东1（杭州）为例，Region填写为oss-cn-hangzhou。
  bucket: "owner-h5",
  region: "oss-cn-shenzhen",
  // 阿里云账号AccessKey拥有所有API的访问权限，风险很高。强烈建议您创建并使用RAM用户进行API访问或日常运维，请登录RAM控制台创建RAM用户。
  accessKeyId: "accessKeyId",
  accessKeySecret: "accessKeyId"
});
const publicPath = path.resolve(__dirname, "./dist");

async function run(proPath = "") {
  const dir = await promisifyReaddir(`${publicPath}${proPath}`);

  for (let i = 0; i < dir.length; i++) {
    const stat = await promisifyStat(path.resolve(`${publicPath}${proPath}`, dir[i]));

    if (stat.isFile()) {
      const fileStream = fs.createReadStream(path.resolve(`${publicPath}${proPath}`, dir[i]));
      console.log(`上传文件: ${proPath}/${dir[i]}`);
      const result = await client.putStream(`${proPath}/${dir[i]}`, fileStream);
    } else if (stat.isDirectory()) {
      await run(`${proPath}/${dir[i]}`);
    }
  }
}

run();
```

打开 deploy.js 将 bucket,region 填入。

在阿里云的 RAM 访问控制 中，创建一个新用户，配置下 oss 管理权限后 Key 和 seccret 复制出来

![img](https://e8ihplx6a2.feishu.cn/space/api/box/stream/download/asynccode/?code=NGI3M2I5MzhlOWNiMGNkOTA3ZTRlZTEyNDFlNWM2MzFfUElMUzdldndReHFqREhWeW1wbmFqR0puNklNSWViR2NfVG9rZW46Ym94Y25DTjBnS0t2ckZFSlRYMzFEeXZ5ZXllXzE2Nzc1NjgzNjk6MTY3NzU3MTk2OV9WNA)

这样再稍微配置下要部署的项目 就可以用这个脚本了。

## **使用**

将 deploy.js 下载到你的项目根目录下。一般是 webpack 打包而成的单页面应用。页面打包生成 dist 文件夹目录，将 dist 文件夹上传到阿里云 oss 上。

在 package.json 中加入这个脚本命令，用来更快的执行部署命令。也可以手动 node deploy.js 执行部署脚本。

```Plaintext
"scripts": {
  "deploy": "node deploy.js"
},
"devDependencies": {
  "ali-oss": "^6.1.1",  // 这是阿里云的oss依赖，也可以直接手动npm install ali-oss --save-dev
}
```
