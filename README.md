# Soda Express Documentation

Soda Express 开发者文档


## 目录结构

```
├── README.md                          // 说明文档
├── custom                             // 文档页面样式及 JavaScript 代码
├── images                             // 文档中引用的所有图片
├── md                                 // 临时目录（文档均为自动生成，因为不要修改）
├── dist                               // 编译之后生成的文件将会在此目录下
├── private                            // 未完成、未发布的文档临时保存在这里，以便让重建全站文档索引的系统任务忽略这些文件
├── react                              // 文档评论功能所需要用的 React 组件
├── server_public                      // 文档评论功能所需要用的 React 组件
├── templates                          // 文档网站的 HTML 页面模板
├── views                              // Markdown 格式文档的模板文件和源文件，使用时会被编译到 md 目录中
├── app.coffee
├── app.json
├── CHANGELOG.md                       // changelog 记录
├── circle.yml
├── CONTRIBUTING.md                    // 贡献指南
├── package.json
└── ...
```

## 预览

开发服务基于 Grunt，所以需要有 Nodejs 环境，通过 NPM 安装测试需要的依赖

安装 Grunt

```bash
$ sudo npm install -g grunt-cli
```

安装需要的依赖

```bash
$ npm ci
```

本地启动一个 HTTP Server，然后打开浏览器访问 <http://localhost:9000> 即可

```bash
$ npm run preview
```

注意保持网络畅通（如有必要请使用代理），因为部分 Native 依赖在网络不畅的情况下无法获取预编译文件，会触发本地编译，耗时较长，也有较大几率会编译失败。

## License

LGPL-3.0
