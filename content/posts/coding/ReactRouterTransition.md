---
title: "React Router Transition"
date: "2022-07-20T15:47:48+08:00"
---

## React Transition

对于React在切换路由的动画转场实现 参考自Vue-Element-Admin

```scss
.window {
  $transitionX : 15%;
  $duration    : 300ms;

  &-enter {
    transform : translate($transitionX, 0);
    opacity   : 0;
  }

  &-enter-active {
    transform  : translate(0, 0);
    opacity    : 1;
    transition : all $duration;
  }

  &-exit {
    transform : translate(0, 0);
    opacity   : 1;
  }

  &-exit-active {
    transform  : translate(-$transitionX, 0);
    opacity    : 0;
    transition : all $duration;
  }
}

```

```tsx


const Window = () => {
    const location = useLocation()
    return <div style={ { width: 600, height: 400, outline: '1px solid black', overflow: 'hidden' } } >
        <SwitchTransition mode="out-in" >
            <CSSTransition
                classNames="window"
                timeout={ 300 }
                key={ location.pathname }
            >
                <Outlet />
            </CSSTransition >
        </SwitchTransition >
    </div >
}


const App = () => {
    const links = [
        'home',
        'login',
        'about'
    ]
    return (
        <HashRouter >
            <Demo />
            <div >
                {
                    links.map(item => {
                        return <div key={ item } ><Link to={ item } >{ item }</Link ></div >
                    })
                }
            </div >
            <Routes >
                <Route path={ '/' } element={ <Window /> } >
                    <Route element={ <About /> } path={ 'about' } />
                    <Route element={ <Login /> } path={ 'login' } />
                    <Route element={ <Home /> } path={ 'home' } />
                    {/* any component */ }
                </Route >
            </Routes >
        </HashRouter >

    )
}
```