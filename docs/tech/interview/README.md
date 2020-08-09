# 登陆拦截逻辑 
第一步：路由拦截
    首先再定义路由的时候就需要多添加一个自定义字段requireAuth，用于判断该路由的访问是否需要登陆，如果用户已经登陆，则顺利进入路由，否则就进入登陆页面
    
    ```
    const routes = [
        {
            path:'/',
            name:'/',
            component:Index
        },
        {
            path:'/repository',
            name:'reponsitory',
            meta:{
                // 添加该字段，表示进入这个路由是需要登陆的
                requireAuth:true
            },
            conponent:Repository
        },
        {
            path:'/login',
            name:'login',
            component:Login
        }
    ]
    ```

    定义完路由后，我们主要是利用vue-router提供的钩子函数beforeEach(),对路由进行判断
    ```
    router.beforeEach((to,form,next)=>{
        if(to.meta.requireAuth) { // 判断该路由是否需要登陆权限
            if(store.state.token) { // 通过vuex state获取当前的token是否存在
                next();
            }else {
                next({
                    path:'/login',
                    query:{redirect:to.fullPath} // 将跳转的路由path作为参数，登陆成功后跳转到该路由
                })
            }
        } else {
            next()
        }
    })
    ```

第二步：拦截器
    要想统一处理所有http请求和响应，就得用上axios的拦截器，通过配置http response inteceptor,当后端接口返回401 Unauthorized(未授权)，让用户重新登陆
    
    ```
    // http request 拦截器
    axios.interceptors.request.use(
        config => {
            if (store.state.token) {
                config.headers.Authorization = `token ${store.state.token}`;
            }
            return config;
        },
        err =>{
            return Promise.reject(err);
        }
    )
    // http response 拦截器
    axios.interceptors.response.use(
        response =>{
            return response;
        },
        error =>{
            if (error.response) {
                switch (error.response.status) {
                    case 401:
                        store.commit(types.LOGOUT);
                        router.replace({
                            path:'login',
                            query:{redirect:router.currentRoute.fullPath}
                        })
                }
            }
            return Promise.reject(error.response.data)
        }
    )
    ```
    种树的最佳时间是十年前和现在。