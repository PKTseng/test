# router
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

---
![](https://i.imgur.com/VJ7bvfv.png)
上面要 import 下面 components 也要引入，這樣才可以使用組件，下面引入後上面模板才可以使用

每個組件只會放在同資料夾路徑下，例如: './xx/xx'
除非用 @ 才會是從最根本的 src 下開始，不然一般組件內的路徑都是從當下那個組件的資料夾內開始往下找