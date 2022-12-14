# NPM (Node Package Manager) Node 的包管理器
是 node 默认的、以 JavaScript 开发的基于 Node.js 的命令行工具，本身也是 node 的一个包

# NPM 安装流程
## 1. 执行安装命令之后，npm 首先会去查找 npm 的配置信息
npm 会在项目中查找是否有 .npmrc 文件，没有的话会再检查全局配置的 .npmrc ，还没有的话就会使用 npm 内置的 .npmrc 文件
## 2. 获取完配置文件之后，就会构建依赖树
首先会检查下项目中是否有 package-lock.json 文件：
存在 lock 文件的话，会判断 lock 文件和 package.json 中使用的依赖版本是否一致，
如果一致的话就使用 lock 中的信息，反之就会使用 npm 中的信息；那如果没有 lock 文件的话，就会直接使用 package.json 中的信息生成依赖树
## 3. 在有了依赖树之后，就可以根据依赖树下载完整的依赖资源
在下载之前，会先检查下是否有缓存资源，如果存在缓存资源的话，那么直接将缓存资源解压到 node_modules 中。
如果没有缓存资源，那么会先将 npm 远程仓库中的包下载至本地，然后会进行包的完整性校验，校验通过后将其添加的缓存中并解压到 node_modules 中
## 4. 会生成 package-lock.json 文件
^3.1.6 意味着安装的时候会安装 3.x.x 的大版本中最新的小版本
lock 这个文件会将本次安装的依赖的版本信息记录下来，在下次再安装的时候，或者其他伙伴使用该包，或者通过 CI 工具的时候，就会安装相同版本的依赖。这样就会避免 package.json 中的内容一致但是实际上安装依赖的版本不一致而造成 Bug 出现的情况。

## 不同NPM版本生成依赖树的区别
* npm 2.X 时期，还是使用的最简单的循环遍历方式，递归地下载所有的依赖包，只要有用到的依赖，都进行安装
* npm 3.X 将原有的循环遍历的方式改成了更为扁平的层级结构，将依赖进行平铺。   
  在生成依赖树的时候，首先会遍历所有的依赖并将其放入树的第一层节点，然后再继续遍历每一个依赖。当有重复的模块时，如果依赖版本相同，就丢弃不放入依赖树中。如果依赖版本不一致，那么就将其放在该依赖下。
* npm 5.X
  在安装依赖流程中的最后一步，会生成 package-lock.json 文件，该文件存储的是确定的依赖信息。也就是说，只要 lock 文件相同，那么每次安装依赖生成的 node_module 就会是相同的。
在项目中使用 package-lock.json 还可以减少安装时间。因为在 package-lock.json 锁文件中已经存放了每个包具体的版本信息和下载链接，这就不需要再去远程仓库进行查询，优先会使用缓存内容从而减少了大量的网络请求
  
# NPM的缓存机制
```shell
npm config get cache # /Users/hanli/.npm
```
* content-v2 文件是用来存在缓存包的具体内容，index-v5 是用来存储依赖包的索引，根据 index-v5 中的索引去 content-v2 中查找具体的源文件 。
* npm 在安装依赖的过程中，会根据 lock文件中具体的包信息，用 pacote:range-manifest:{url}:{integrity} 规则生成出唯一的 key，然后用这个 key 通过 SHA256 加密算法得到一个 hash。这个 hash 对应的就是 _cache/index-v5 中的文件路径，前 4 位 hash 用来区分路径，剩下的几位就是具体的文件名。文件的内容就是该缓存的具体信息了

1. 在安装资源的时候，npm 会根据 lock 中的 integrity、version、name 信息生成一个唯一的 key。
2. 然后用这个 key 经过 SHA256 算法生成一个 hash，根据这个 hash 在 index-v5 中找到对应的缓存文件，该缓存文件中记录着该包的信息。
3. 根据该文件中的信息我们在 content-v2 中去找对应的压缩包，这样就找到了对应的缓存资源了。
4. 最后再将该压缩包解压到 node_modules 中。
5. 通过资源包中的 shasum 校验完整性。

## 经验之谈
1. 尽量使用 npm v5.7 以上版本
2. 建议在项目中使用 package-lock.json 文件，并将其提交到远端仓库，从而提升依赖安装时间以及安装包的稳定性
3. 如果 package-lock.json 文件冲突，应该先手动解决 package.json 的冲突，然后执行 npm install --package-lock-only，让 npm 自动帮你解冲突

