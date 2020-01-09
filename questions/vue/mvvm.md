# 解释mvc mvvm mvp

> 通过分层，将职责分离开。

## mvc (单向通信)
> 用户与 View 交互，触发用户事件 (监听Model数据变化)
>
> Controller 监听到用户事件，调用model接口，改变Model层数据 (依赖Model和View)
>
> Model 数据变化，将数据传给View层，View更新

## mvp (Presenter双向通信, V和M之间不可见)
> 用户与 View 交互，触发用户事件
>
> Presenter 监听到用户事件，调用 Model 接口，改变 Model 层数据
>
> Mdoel数据变化，触发相应事件，Presenter监听到后调用View的接口，View更新

## mvvm(与mvp相似，区别是v和vm之间的data-binding)
> 用户与 View 交互，触发用户事件
>
> ViewModle依赖于Model，监听Model事件
>
> View的变化会自动更新到ViewModel，ViewModel的变化也会自动同步到View上显示。ViewModel中的属性实现了Observer

