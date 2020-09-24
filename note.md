# router 0924
```javascript=
export const constantRoutes = [
  {
    path: '/redirect',
    component: Layout,
    hidden: true,
    // hidden 作用?
    // 有些頁面要隱藏，但是又必須存在，
    // 例如 404 頁面，必須存在但又只能在錯誤時顯示
    children: [
      {
        path: '/redirect/:path(.*)',
        // 歐文電影例子，一個組件可以重複使用，但會有很多個 id ，
        // 每個 id 都會對應到不同電影，同時 id 也是跟後端 api 溝通的橋樑
        // 怎麼點都會回到首頁?
        component: () => import('@/views/redirect/index')
        // 這樣的寫法等於上面 import 寫法?
      }
    ]
  },
```
[id 解釋](https://medium.com/@linwei5316/vue-router-4c2aad1cc352)

---
原作命名:
![](https://i.imgur.com/PKR0QpI.png)

Affix 是指固定在某個地方

---
![](https://i.imgur.com/VJ7bvfv.png)

上面要 import 下面 components 也要引入，這樣才可以使用組件，下面引入後上面 template 模板才可以使用

每個組件只會放在同資料夾路徑下，例如: './xx/xx'
除非用 @ 才會是從最根本的 src 下開始，不然一般組件內的路徑都是從當下那個組件的資料夾內開始往下找

---
![](https://i.imgur.com/IHWSfOZ.png)
router ，最上面的 constantRoutes 是一旦命名就不能再更動的，像 ES6 的 cosnt 語法，第二個 asyncRoutes 是依照登入密碼的權限來決定我可以更動哪些東西，例如
![](https://i.imgur.com/2ILLH1J.png)

這權限是 'Admin', 'Editor' 可以登入的，

看下一張
![](https://i.imgur.com/ls1XNy9.png)
這就是 Viewer 最高權限只能到這邊，但是上一張那層的權限就不行
簡單講 roles 有寫到才有那個權限，沒寫到就沒有
以下為上面兩張圖的部分代碼
```javascript=
export const asyncRoutes = [
  {
    path: '/discover',
    component: Layout,
    children: [
      {
        path: 'index',
        component: () => import('@/views/discover/index'),
        name: 'Discover',
        meta: {
          title: 'discover',
          icon: 'discover',
          roles: ['Admin', 'Editor']
        }
      }
    ]
  },
  {
    path: '/information',
    component: Layout,
    redirect: '/information/server',
    name: 'Information',
    meta: {
      title: 'information',
      icon: 'servers'
    },
    children: [
      {
        path: 'server',
        component: () => import('@/views/information/server/index'),
        name: 'Server',
        meta: {
          title: 'server',
          icon: 'servers',
          roles: ['Admin', 'Editor', 'Viewer']
        },
        children: [
          {
            path: 'detail',
            component: () => import('@/views/information/server/detail/index'),
            name: 'Detail',
            hidden: true,
            meta: {
              title: 'detail',
              activeMenu: '/information/server',
              roles: ['Admin', 'Editor', 'Viewer']
            },
```

---
computed 屬性是函式執行完會把變數值 return 出來並丟給 template 模板使用，會把值 return 出來是因為我們不可能在模板裡面加入太多的運算，在模板裡面運算也很難維護，所以就要用 computed 把值 return 出來

methods 是函示執行完裡面的便是同時消失，不會把變數結果 return 出來

[參考文章](https://cythilya.github.io/2017/04/15/vue-computed/)