#Vite

## 为什么是Vite？
1. 在浏览器支持ES模块之前，JS并没有提供原生机制让开发者以模块化方式进行开发。
2. webpack,rollup和Parcel等工具的改善了开发体验，但是遇到了性能瓶颈,文件修改后的效果也需要几秒钟才能在浏览器中反映出来。
3. Vite 旨在利用生态系统中的新进展解决上述问题：浏览器开始原生支持 ES 模块，且越来越多 JavaScript 工具使用编译型语言编写。

## 缓慢的服务器启动
当冷启动开发服务器时，基于打包器的方式启动必须优先抓取并构建你的整个应用，然后才能提供服务。
