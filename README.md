# WebAssembly

## DEMO

```js
WebAssembly.compile(new Uint8Array(`
  00 61 73 6d  01 00 00 00  01 0c 02 60  02 7f 7f 01
  7f 60 01 7f  01 7f 03 03  02 00 01 07  10 02 03 61
  64 64 00 00  06 73 71 75  61 72 65 00  01 0a 13 02
  08 00 20 00  20 01 6a 0f  0b 08 00 20  00 20 00 6c
  0f 0b`.trim().split(/[\s\r\n]+/g).map(str => parseInt(str, 16))
)).then(module => {
  const instance = new WebAssembly.Instance(module)
  const { add, square } = instance.exports

  console.log('2 + 4 =', add(2, 4))
  console.log('3^2 =', square(3))
  console.log('(2 + 5)^2 =', square(add(2 + 5)))

})
```

## wasm是什么

wasm是可以在浏览器中运行的二进制模块。

wasm可以通过由c或c++（python等其他语言也可以）编写的代码文件编译得到。

通过wasm可以在浏览器运行由其他语言编写的代码，不再局限于js。

可以使用其他语言所具有的特性在浏览器上进行开发。

```bash
	Clang		Fastcomp	Binaryen
C/C++ ===> LLVM IR ===> asm.js ===> wasm
```

## wasm的教程

> 参考文章：
>
> https://segmentfault.com/a/1190000008402872
>
> https://blog.csdn.net/TurkeyCock/article/details/83317935
>
> https://www.jianshu.com/p/1fd5496a0971?utm_source=oschina-app
>
> https://www.jianshu.com/p/0bc430ef612d?from=groupmessage
>
> https://zhuanlan.zhihu.com/p/42955781

1. 安装emsdk的依赖

   - python：安装emsdk的脚本为python，运行emsdk命令的也是python
   - git：用于抓取安装emsdk所需的库
   - CMake：为emsdk提供创建makefile文件的功能（即emconfigure）

   - 编译工具： 在linux下安装GCC；在mac下安装XCode；在Windows下安装VS（应该是给emsdk的编译提供IDE，即emmake）

2. 安装emsdk

   ```bash
   git clone https://github.com/juj/emsdk.git
   cd emsdk
   .\emsdk install latest # 安装环境
   .\emsdk activate latest # 激活环境
   .\emsdk_env.bat # 注入环境配置
   ```

   > 在激活环境和注入环境配置后就可以使用upstream/emscripten目录下的指令，比如emcc、emconfigure、emmake等命令。

3. 测试是否安装成功

   ```
   emcc -v
   ```

4. 使用emsdk生成wasm

   ```
   emcc example.c -Os -s WASM=1 -s SIDE_MODULE=1 -o example.wasm
   ```

5. 生成makefile文件和编译

   ```bash
   emconfigure ./configure
   # 根据configure文件生成makefile文件
   # emconfigure是指用emsdk的构建器
   emmake make
   ```

## 在js中加载和使用wasm的方法

```js
/**在 js 中加载 wasm
 * @param {String} path wasm 文件路径
 * @param {Object} imports 传递到 wasm 代码中的变量
 */
function loadWebAssembly (path, imports = {}) {
  return fetch(path)
    .then(response => response.arrayBuffer())
    .then(buffer => WebAssembly.compile(buffer))
    .then(module => {
      imports.env = imports.env || {}

      // 开辟内存空间
      imports.env.memoryBase = imports.env.memoryBase || 0
      if (!imports.env.memory) {
        imports.env.memory = new WebAssembly.Memory({ initial: 256 })
      }

      // 创建变量映射表
      imports.env.tableBase = imports.env.tableBase || 0
      if (!imports.env.table) {
        // 在 MVP 版本中 element 只能是 "anyfunc"
        imports.env.table = new WebAssembly.Table({ initial: 0, element: 'anyfunc' })
      }

      // 创建 WebAssembly 实例
      return new WebAssembly.Instance(module, imports)
    })
}

// 在 js 中使用 wasm
loadWebAssembly('./math.wasm')
  .then(instance => {
    const add = instance.exports._add
    const square = instance.exports._square

    console.log('2 + 4 =', add(2, 4))
    console.log('3^2 =', square(3))
    console.log('(2 + 5)^2 =', square(add(2 + 5)))
  })


// 给 wasm 传递 Web API
const imports = {
  Math,
  objects: {
    count: 2333
  },
  methods: {
    output (message) {
      console.log(`-----> ${message} <-----`)
    }
  }
}

loadWebAssembly('path/to/source.wasm', imports)
  .then(instance => {
    // ...
  })
```

## 使用Anaconda

1. 查看版本信息：conda info

2. 创建python版本为3.9且名字标识为exampleName的环境：conda create -n exampleName python=3.9

3. 激活名为exampleName的环境：conda activate exampleName

4. 退出当前环境：conda deactivate

5. 删除名为exampleName的环境：conda remove -n exampleName --all

6. 将anaconda切换到32位（须管理员）：set CONDA_FORCE_32BIT=1

7. 在32位anaconda中下载的python为32位的版本。

8. 将shell初始化为conda命令行环境：conda init

9. 环境配置（给shell添加conda命令）：在PATH中添加Anaconda中Scripts目录的路径。

   > 在在PATH中添加Anaconda目录的路径可以使得不在conda环境下也可以使用python命令。
   >
   > PATH就是用于告诉命令对应的执行程序要到哪里去找。


## Makefile教程

https://blog.csdn.net/weixin_38391755/article/details/80380786/