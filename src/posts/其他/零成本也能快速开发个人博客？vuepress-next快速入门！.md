---
date: 2022-11-20
category:
  - 前端
tag:
  - blog
archive: true
---

# 零成本也能快速开发个人博客？vuepress-next快速入门！

> 通过本篇文章，你能够知道vuepress是什么，如何进行简单的开发，如何使用Vercel部署，并且使用自己的域名来访问这个网站。

### 原来零成本也能开发个人博客

首先，本篇文章说的“零成本”，说的是我们不需要用额外的精力去学习后端开发（因为我们生成的是一个静态站点）。不需要额外去购买服务器（本篇文章我们使用的Vercel部署）。以及你可以选择是否购买域名并且在站点中使用它。

在刚开始学前端的一段时间里，我都以为开发个人博客需要自己写前端和后端，然后租个服务器和域名。我之前还花很多时间做了个简陋的博客（由于当时刚开始学，做出来的东西又丑又不好用，所以这里就不放项目了）。但是后来发现自己用Typora在本地写写也不错（因为写的都是很基础的东西😂，也不好意思拿出来分享）。

随着时间的推移，我发现，原来还有像Hexo这样的库。也可以通过简单的开发搭建博客。不过后来我发现还有语雀这样的平台，感觉在语雀上面发表博客也不错，同样也有社交属性。但是看到好多大佬都有自己的个人博客，我也有点开始蠢蠢欲动。

说干就干！首先，在进行开发前，我们要进行“技术选型”。在进行综合评定后，我选择了**vuepress-next（也就是vuepress-v2）**

### 为什么要使用vuepress-next？

在官方文档中，也通过与竞品的比较，说明了选择vuepress-next的好处：[为什么不是...?](https://v2.vuepress.vuejs.org/zh/guide/#%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E6%98%AF)

而对于我来说，我平常开发使用vue3，所以我一开始就将眼光放到了vuepress-next和vitepress上。这两者都是以**内容为中心**，不过vitepress的可配置性更低。我认为作为一个个人博客，虽然不用太花里胡哨，但个性化还是需要的。

所以最后我选择了vuepress-next。

> 不过需要注意的是，在写本篇文章时，vuepress-next还是测试版本，最新版本为v2.0.0-beta.53。

### 什么是vuepress？

首先，VuePress 是一个以 Markdown 为中心的静态网站生成器。vuepress内置了[markdown-it](https://github.com/markdown-it/markdown-it)，通过vuepress，你的md文件将会被编译成html页面。由于vuepress是基于vue的，所以你也可以在md文件中使用vue语法。

接下来我将通过项目的搭建，以及一些简单的配置来带你了解vuepress。

### 搭建项目

搭建项目虽然是入门一个库最基本的操作，但最开始由于没认真读文档，我还踩了两个坑。这两个坑会在下文提到。

首先新建一个文件夹，进入这个文件夹，开始搭建项目：

在这篇文章，我们使用pnpm来作为我们的包管理器。

-   生成package.json文件

```
pnpm init
```

-   安装依赖

```
pnpm install -D vuepress@next
```

**需要注意的是，使用 pnpm时，你可能需要安装 `vue` 和 `@vuepress/client` 作为 peer-dependencies ，即：**

```
pnpm add -D vue @vuepress/client@next
```

上面那句话是文档的原话，当时看到下载的包都是`next`标签，误以为vue也是用的next，但实际上人家文档写得清清楚楚。（我们需要注意vue，vuepress，vuepress/client都为最新版本，当前为vue^3.2.45），**如果peer-dependencies的版本不一致会导致报错：[useXXX() is called without provider](https://vuepress-theme-hope.github.io/v2/zh/faq/common-error.html#usexxx-is-called-without-provider)**（所以大家在读文档的时候别跳着看，避免浪费时间）



-   在package.json文件中添加启动和打包指令

```
"scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
```

-   生成`.git`文件夹，并且添加`.gitignore`文件

```
git init
```

创建`.gitignore`文件，并写入：

```
/node_modules
.temp
.cache
```

-   新建文件夹docs，并在docs目录下新建README.md文件，在其中输入：

```
## vuepress-demo
```

然后打开链接，你将会得到这样一个页面。

![image-20221121225519888](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3616e28e7b43409abf79a06ddc579015~tplv-k3u1fbpfcp-zoom-1.image)

**其中对于md文件，需要注意的是，如果你的md文件中有`<>`这种未闭合的标签，请使用反引号（`）包裹。因为md文件再vuepress中会被编译为html文件，如果是一个未闭合的标签，则会报错。这也是我踩的第二个坑。**




> 在上面的步骤中，我们搭建了一个最基础的项目。现在我们来配置一下我们的首页

### Frontmatter配置

Markdown 文件可以包含一个 [YAML在新窗口打开](https://yaml.org/) Frontmatter 。Frontmatter 必须在 Markdown 文件的顶部，并且被包裹在一对三短划线中间。下面我们将会配置一个示例首页：[Frontmatter的其他配置可以看这个](https://v2.vuepress.vuejs.org/zh/reference/frontmatter.html)

```
---
home: true #是否为首页（这里这是看是否使用首页的样式，我们可以有很多md文件的home为true）
heroText: Kevin Qian
heroImage: /pic/R-c.jpg #页面图片（这里的/就是/public）
tagline: 软件工程，没有银弹。 #首页的标语。
features: #配置首页特性列表。
  - title: 前端
    details: js|ts|vue3。
  - title: 后端
    details: nodejs|nestjs
  - title: 随记
    details: 一些想法
actions:
  - text: 进入我的博客 #按钮的文字
    link: /blogs #跳转页面
    type: secondary #按钮的type
---
```

我们在docs文件夹下创建public文件夹，然后创建pic文件夹，将页面图片放入其中。

这是目前的文件夹：

![image-20221121231522178](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9821946f3eac4ea1a0f0c9bd97e151ab~tplv-k3u1fbpfcp-zoom-1.image)

然后我们得到了这样的页面：

![image-20221121231657697](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b91b94f61d745159b2d09126d783714~tplv-k3u1fbpfcp-zoom-1.image)

### 配置文件：config.ts文件

当然，我们不可能对一个个的md文件单独的配置，所以我们需要一个全局的配置文件。

我们在`.vuepress`文件夹下创建`config.ts`文件，使用ts文件我们能得到更好的类型提示，不熟悉ts的朋友可以去看看我的**ts入门系列文章**：[TS入门小记 - 跟我一起秃秃秃的专栏 - 掘金 (juejin.cn)](https://juejin.cn/column/7163571163137277965)

我们先将首页左上角的网站名写上，然后弄一个导航栏：

在此之前，我们需要安装@vuepress/theme-default：

```
pnpm i -d @vuepress/theme-default@next 
```

在写配置文件前，由于我们会用到一个我们的一个md文档，我们在doc目录下新建一个文件夹`backend`，并新建一个文件：`1_nest.md`

```
import { defineUserConfig } from "vuepress";
import { defaultTheme } from "@vuepress/theme-default";

export default defineUserConfig({
  lang: "zh-CN",
  title: "Kevin Qian", // 显示在左上角的网页名称以及首页在浏览器标签显示的title名称
  description: "Kevin Qian blog", // meta 中的描述文字，用于SEO
  plugins: [],
  // 默认主题
  theme: defaultTheme({
    repo: "https://github.com/xxx/xxx", //github仓库地址
    editLink: false,
    lastUpdatedText: "最后更新时间",
    contributorsText: "作者",
    navbar: [
      // NavbarItem
      {
        text: "前端",
        link: "/frontend",
      },
      // NavbarGroup
      {
        text: "后端",
        children: ["/backend/1_nest.md"],
      },
      // 字符串 - 页面文件路径
      {
        text: "算法",
        children: [],
      },
      {
        text: "软件工程学",
        children: [],
      },
    ],
  }),
});

```

首页：

![image-20221121232928372](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a67a7170ff934794b3cac7d8c4cd31ee~tplv-k3u1fbpfcp-zoom-1.image)

这是我们在“后端”导航中的一个子链接跳转后的页面：

![image-20221121233636573](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/528fd663b85e4575a1e6f737b2939330~tplv-k3u1fbpfcp-zoom-1.image)

### 路由

通过上面的实践，我们可以发现页面的根地址为docs文件夹下的`README.md`文件，（index.md）也行。然后`docs/backend/1_nest.md`对应的是`/backend/1_nest.html`。关于路由，可以看官方文档：[路由](https://v2.vuepress.vuejs.org/zh/guide/page.html#%E8%B7%AF%E7%94%B1)

### 添加搜索插件和代码块复制插件

我们的博客现在还没有文章列表，文章列表会在该栏目的下一篇文章中写道：[vuepress - 跟我一起秃秃秃的专栏 - 掘金 (juejin.cn)](https://juejin.cn/column/7168496265888530463)，所以我们现在要找到我们的文章可以耍个小花招（反正之后也会添加这两个插件），添加搜索插件。

我们需要安装一个社区插件，我们顺便把代码块复制插件也装上：

```
pnpm i -d @vuepress/plugin-search@next vuepress-plugin-copy-code2@next
```

然后再config.ts中的plugins中使用这两个插件：

```
plugins: [
    // 搜索插件
    searchPlugin({
      hotKeys: ["ctrl", "k"], //聚焦热键为ctrl+k
    }),
    // 复制代码插件
    copyCodePlugin({
      showInMobile: true, //是否显示在移动端
      pure: true, //复制按钮在代码块右上角
    }),
  ],
```

然后我们运行项目：

![image-20221121235435199](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fccb473803e94f14bff61eb745c901f2~tplv-k3u1fbpfcp-zoom-1.image)

![image-20221121235503172](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ad544e77ab14fda9be7b613bc93ef69~tplv-k3u1fbpfcp-zoom-1.image)



### 如何使用vercel部署？

可以参考这篇文章：[我选择白嫖！使用vercel快速部署vuepress-next项目 - 掘金 (juejin.cn)](https://juejin.cn/post/7168783748048093215/)