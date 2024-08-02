![image](https://github.com/user-attachments/assets/67781cc1-435c-4223-8f8a-d42d40a2019d)
## 1.功能

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731231242391.png" alt="image-20240731231242391" style="zoom:50%;" />

## 2.⚠️⚠️分包

如果站点中的所有依赖都打包到一个js文件中，势必会导致打包结果过大

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731231522153.png" alt="image-20240731231522153" style="zoom: 33%;" />

⚠️⚠️而实际上，在页面初始的时候，不需要那么多代码参与运行。

比如在这个项目中，一开始必须要运行的只有封面模块，因为它是用户一开始就必须要能够看见的。而电影模块可以慢慢加载。

基于此，我们可以使用==**动态导入的方式加载电影模块**==

```js
// main.js
import './cover'; // 静态导入，表示初始就必须要依赖 cover 模块

// movie 模块
import('./movie') // 动态导入，表示运行到此代码时才会去远程加载。这里返回的是一个promise
```

webpack能够识别动态导入的代码，当它发现某个模块是==**使用动态导入时，该模块会单独形成打包结果**==

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731231720867.png" alt="image-20240731231720867" style="zoom:33%;" />

在浏览器运行时，会**首先加载初始的打包结果，然后在后续的运行过程中，动态加载其他模块**。这样就可以尽量提升初始加载效率，又不影响后续模块的加载

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731231752407.png" alt="image-20240731231752407" style="zoom:33%;" />

## 3.⚠️⚠️⚠️跨域代理

**大部分时候，为了安全，服务器都是不允许跨域访问的**

 所以，将来**部署应用的时候，通常会使用下面的方式**进行部署

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731232204592.png" alt="image-20240731232204592" style="zoom: 50%;" />

你无须彻底理解上图，只需要知道：==**最终部署之后，不存在跨域问题**==

但是，==跨域问题在开发阶段是存在的==！

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731232105012.png" alt="image-20240731232105012" style="zoom: 50%;" />

所以，我们要做的，**仅仅是消除开发阶段的跨域问题**，便于在开发阶段查看效果

如何实现：

1.在==webpack.config.js==中，找到下面的部分，设置代理

```js
devServer: {//配置开发服务器
  proxy: {
    '/api': { // 当请求地址以 /api 开头时，代理到另一个地址
      target: 'http://study.duyiedu.com', // 代理的目标地址
      changeOrigin: true, // 更改请求头中的host，为避免出问题，最好写上
    }
  }
}
```

> **`proxy`**: 这个属性用来配置一个或多个代理规则，帮助你将特定的请求代理到另一个服务器。
>
> **`'/api'`**: 这是一个路径匹配模式，意味着==所有以 `/api` 开头的请求都会被代理到指定的 `target`==。
>
> **`target`**: 这是代理请求的目标地址，即实际的服务器地址。
>
> **`changeOrigin`**: 设置为 `true`，代表会修改 HTTP 请求头中的 `Host` 为目标地址。这对于一些检查 `Host` 头的服务器来说是必要的，可以帮助避免跨域问题。

2.在ajax请求时，仅需给上请求路径即可

```js
axios.get(http://study.duyiedu.com/api/movies');//❌无须指定源
axios.get('/api/movies');//✅
```

来看看这样做的效果是什么

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731232750433.png" alt="image-20240731232750433" style="zoom: 50%;" />

这样依赖，在跨域问题上，就做到了开发环境与生产环境的统一

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731232845121.png" alt="image-20240731232845121" style="zoom: 33%;" />

## 4.电影模块

<img src="C:\Users\13890\AppData\Roaming\Typora\typora-user-images\image-20240731232904273.png" alt="image-20240731232904273" style="zoom: 50%;" />

### list模块

该模块很简单，按照如下思路实现即可

```js
import $ from 'jquery';
import styles from './index.module.less';

let container;
/**
 * 初始化函数，负责创建容器
 */
function init() {
  container = $('<div>').addClass(styles.container).appendTo('#app');
}

init();

/**
 * 根据传入的电影数组，创建元素，填充到容器中
 * @params movies 电影数组
 */
export function createMovieTags(movies) {
  const result = movies
    .map(
      (m) => `<div>
  <a href="${m.url}" target="_blank"><img src="${m.cover}"></a>
  <a href="${m.url}" target="_blank"><p class="${styles.title}">${m.title}</p></a>
  <p class="${styles.rate}">${m.rate}</p>
  </div>`
    )
    .join('');
  container.html(result);
}

```

```less
@itemWidth: 150px;

.container {
  width: 1000px;
  margin: 30px auto;
    /*网格布局*/
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  text-align: center;
  color: #555;
  font-size: 14px;
  row-gap: 30px;
  img {
    width: @itemWidth;
    height: 200px;
  }
}
.title {
  margin-top: 10px;
  margin-bottom: 5px;
}
.rate {
  font-size: 12px;
  color: #e4a64a;
}

```



### ⚠️⚠️⚠️pager模块

#### createPagers

该函数的实现可以按照下面的思路进行

```js
import $ from 'jquery';
import styles from './index.module.less';
import { getMovies } from '@/api/movie';
import { createMovieTags } from '@/movie/list';

let container;
/**
 * 初始化函数，负责创建容器
 */
function init() {
  container = $('<div>').addClass(styles.pager).appendTo('#app');
}

init();

/**
 * 根据传入的页码、页容量、总记录数，创建分页区域的标签
 * @params page 页码
 * @params limit 页容量
 * @params total 总页数
 */
export function createPagers(page, limit, total) {
  container.empty();
  /**
   * 辅助函数，负责帮忙创建一个页码标签
   * @params text 标签的文本
   * @params status 标签的状态，⚠️空字符串-普通状态，disabled-禁用状态，active-选中状态
   */
  function createTag(text, status, targetPage) {
    const span = $('<span>').appendTo(container).text(text);
    const className = styles[status];
    span.addClass(className);
    // 只有是普通样式时才需要监听点击事件
    if (status === '') {
      span.on('click', async function () {
        //1. 重新拿数据
        const resp = await getMovies(targetPage, limit);
        //2. 重新生成列表
        createMovieTags(resp.data.movieList);
        //3. 重新生成分页区域
        createPagers(targetPage, limit, resp.data.movieTotal);
      });
    }
  }
  const pageNumber = Math.ceil(total / limit); // 最大页码 向上取整
  //1. 创建首页标签
  createTag('首页', page === 1 ? 'disabled' : '', 1);
  //2. 创建上一页标签
  createTag('上一页', page === 1 ? 'disabled' : '', page - 1);
  //3. 创建数字页码标签
  const maxCount = 10; // 定一个最大数字页码的数量
  let min = Math.floor(page - maxCount / 2);
  min < 1 && (min = 1); //如果min<1就把min变为1
  let max = min + maxCount - 1;
  max > pageNumber && (max = pageNumber);
  for (let i = min; i <= max; i++) {
    createTag(i, i === page ? 'active' : '', i);
  }

  //4. 创建下一页标签
  createTag('下一页', page === pageNumber ? 'disabled' : '', page + 1);
  //5. 创建尾页标签
  createTag('尾页', page === pageNumber ? 'disabled' : '', pageNumber);
}

```

