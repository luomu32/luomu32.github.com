# NodeJS

NodeJS是基于V8引擎的JavaScript开发环境。最大的特点是采用异步模型，提升吞吐量。

网络IO中，底层Linux系统依赖Epoll，Windows系统依赖IOCP。

## 包管理NPM

npm init命令用于初始化package.json文件。

npm run命令用于执行package.json文件中script部分定义的脚本。

npm install -g全局安装。npm install = npm i=npm install --save，npm install --no-save表示临时引用，不加入package.json中的dependencies。npm install --save-dev

## 测试

Mocha测试框架，支持TDD和BDD。

代码覆盖率工具，Istanbul

## 模块

