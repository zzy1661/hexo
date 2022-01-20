---
title: webpack5 升级踩坑
date: 2022-01-19 10:52:17
tags:
---
1. 安装webpack@5
    -   `yarn add webpack@^5.36.2 webpack-cli@^4.6.0`
2. 解决安装过程中提示的版本不兼容warning

    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f89dd471a85f4238949f7c0850cf121f~tplv-k3u1fbpfcp-watermark.image)

3. 处理过时的loader、plugin和配置
    - 移除cache-loader
    - 移除friendly-errors-webpack-plugin
    - optimize-css-assets-webpack-plugin 替换为css-minimizer-webpack-plugin
    - TerserPlugin去掉了sourcmap和cache
    - 删除 new webpack.optimize.ModuleConcatenationPlugin()
    - 删除eslint-loader,使用`ESLintPlugin`
    - 废弃了 raw-loader，url-loader 和 file-loader,新配置参考如下
        ```
        {
				test: /\.(png|jpe?g|gif|svg|eot|woff|woff2|ttf)(\?.*)?$/,
				exclude: /(antd)/,
				type: 'asset',
				parser: {
					dataUrlCondition: {
						maxSize: 10 * 1024 // 10kb
					}
				},
				generator: {
					filename: `${config.ChunkOutputDirname}assets/img/[name].[hash:7].[ext]` //'static/[hash][ext][query]'
				}
		
			}
        ```
4. 根据文档挨个检查处理
    
    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a9e90766352487e9445f0db4706ff0c~tplv-k3u1fbpfcp-watermark.image)

5. 常见的报错
    
    - dev server 启动报错
    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e01b6f56c7ee4723811457dc17ac563c~tplv-k3u1fbpfcp-watermark.image)
    解决： 通过`webpack serve`启动dev server
        ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dd4d8486dbf41c5bb421fec5a1a3b39~tplv-k3u1fbpfcp-watermark.image)
        
    - plugin引用错误

    ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e77c54098c845749ec0c60ee14eeb74~tplv-k3u1fbpfcp-watermark.image)
    解决： 部分plugin写法变更 `const {WebpackManifestPlugin} = require('webpack-manifest-plugin');`
    - 打包后文件运行报错Uncaught TypeError: Cannot read property 'dispose' of undefined
    解决：dev包和本地同时启用了缓存，并缓存保存在同一个目录时会出现混乱，需要将不同环境的缓存区分（生产环境禁止使用缓存）

      ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/914e69f427bc412a90384306391d10bf~tplv-k3u1fbpfcp-watermark.image)
    - webpack 5 运行于 Node.js v10.13.0+ 的版本，如果出现打包错误，可以检查版本
    - 如果配置了splitchunk，HtmlWebpackPlugin的chunks和chunksSortMode需要注意顺序，最好让它自己决定chunks和sort
