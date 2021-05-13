taro 项目

修改config/index.js
开启css modules

ts文件引入报错
根目录增加typings.d.ts
写入以下内容
declare module '*.css';
declare module '*.less';
declare module '*.scss';
declare module '*.sass';
declare module '*.svg';
declare module '*.png';
declare module '*.jpg';
declare module '*.jpeg';
declare module '*.gif';