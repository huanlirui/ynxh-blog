部署Hexo踩过的坑—node14.0配置hexo

hexo -v出现的问题


(node:5749) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
(node:5749) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:5749) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:5749) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:5749) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:5749) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
hexo: 4.2.0
hexo-cli: 3.1.0
os: Darwin 18.7.0 darwin x64
node: 14.16.0
v8: 8.4.371.19-node.18
uv: 1.40.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.16.1
modules: 83
nghttp2: 1.41.0
napi: 7
llhttp: 2.1.3
openssl: 1.1.1j
cldr: 37.0
icu: 67.1
tz: 2020a
unicode: 13.0


hexo d -g 出现的问题


Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
(node:5183) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:5183) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:5183) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:5183) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:5183) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
TypeError [ERR_INVALID_ARG_TYPE]: The "mode" argument must be integer. Received an instance of Object
    at copyFile (fs.js:1972:10)
    at tryCatcher (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/util.js:16:23)
    at ret (eval at makeNodePromisifiedEval (/usr/local/lib/node_modules/hexo-cli/node_modules/bluebird/js/release/promisify.js:184:12), <anonymous>:13:39)
    at /Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/hexo-fs/lib/fs.js:144:39
    at tryCatcher (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:547:31)
    at Promise._settlePromise (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:604:18)
    at Promise._settlePromise0 (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:649:10)
    at Promise._settlePromises (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:729:18)
    at Promise._fulfill (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:673:18)
    at Promise._resolveCallback (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:466:57)
    at Promise._settlePromiseFromHandler (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:559:17)
    at Promise._settlePromise (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:604:18)
    at Promise._settlePromise0 (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:649:10)
    at Promise._settlePromises (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:729:18)
    at Promise._fulfill (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:673:18)
    at Promise._resolveCallback (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:466:57)
    at Promise._settlePromiseFromHandler (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:559:17)
    at Promise._settlePromise (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:604:18)
    at Promise._settlePromise0 (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:649:10)
    at Promise._settlePromises (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:729:18)
    at Promise._fulfill (/Users/getaikejiff/Desktop/hlr/ynxh-blog/myblog/node_modules/bluebird/js/release/promise.js:673:18)




    出现这些是因为node版本太高，切换成低版本的node来安装Hexo就可以了

我原先是安装了最新版node14.0，后来多装了一个比较稳定的node12.14版本，这个问题就解决了




似乎是由hexo-fs引起的。hexo-deployer-git取决于hexo-fs，hexo-fs @ 2.0.0与Node.js 14不兼容。

我们已经发布hexo-fs@2.0.1了支持Node.js 14的版本。
请重新安装hexo-deployer-git吗？

我认为，重新安装后hexo d会很好用。