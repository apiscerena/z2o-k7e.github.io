-----

[TOC]

-----

## 贡献流程

1. Github 上 fork 本 [Repo](https://github.com/ZKPunk-Org/ZKPedia)
2. 在 `./src/zk-everything` 下 `mkdir` 一个以自己名字命名的文件夹
3. `src/SUMMARY.md` 是前端网站显示的文件组织目录，修改该文件，找到一个合适的放置目录，将文章的本地 `.md` 文件位置链接过去
4. 提交 Pull Request
5. Pull Request 被 merge 后很快就会在 <https://learn.zkpunk.pro> 网站显示

## 文章格式

#### 1. 文章元数据

可以添加一些文章的元数据, 如 「作者 (required）」，「标签、联系方式 (optional） 」

```bash
> 作者: 如 @Alice https://github.com/Alice
> 标签: 如 Halo2, Nova, STARK, Folding Schema .... # mdbook 暂不支持 tag 功能
> 时间: 2024-10-01
```

比如：
> 作者：[Alice](https://github.com/alice)


#### 2. 文章目录

文章开始之前，可以添加 `[TOC] `来让 mdbook 自动生成该文章的 Table of contents（目录）

```text
-----

[TOC]

-----
```

#### 3. 文章正文

按照常规的 Markdown 格式去写就好

如果想要高亮显示某段文字, 可以使用 [mdbook-admonish](https://tommilligan.github.io/mdbook-admonish/), 比如:

```admonish success title=""
This will take a while, go and grab a drink of water.
```

如何添加图片？

 - 推荐直接在 `.md` 文章同级目录 `mkdir ./imgs` 目录，mdbook 中直接引用该 imgs 目录相对路径
 - 如果使用的是 OSS 云存储，则无需考虑图片存储，只需一个 `.md` 文件即可


#### 4. 文章末尾

可以列出 「致谢」 & 「参考文献」

```
# References
 - [trapdoor-tech halo2 book](https://trapdoor-tech.github.io/halo2-book-chinese/user/simple-example.html)
 - [icemelon/HaiCheng Shen](https://github.com/icemelon/halo2-examples/blob/master/src/fibonacci/example3.rs)
 - [0xPARC halo2](https://learn.0xparc.org/)
```

----


## 关于 markdown 渲染

众所周知，Github  网站的 `Latex` 渲染功能非常弱，往往需要一些技巧才能让公式正常渲染。我们使用的 mdbook，不需要给 github markdown 渲染做专门的适配和魔改, 主流 Markdown 编辑器（如 Obsidian, Typora）里的 `.md` 文件都可以正常渲染

**本地 Dev 预览方法：**

1. [安装 Rust](https://www.rust-lang.org/tools/install)

2. 安装依赖

```bash
cargo install mdbook mdbook-latex  mdbook-toc
```

3. 运行

```bash
mdbook serve --open 
```

Tips :
 - `src/SUMMARY.md` 会在前端显示所有文件目录及其链接
 - 公式测试：可以在 [katex.org](katex.org) 测试. 如果使用 Obisidian notes 编辑公式, 这里不需要任何修改

