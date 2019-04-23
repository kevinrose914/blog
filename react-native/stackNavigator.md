### 先看用法
路由相关的js
```js
import { createStackNavigator } from 'react-navigation'
import React from 'react'
import { Button } from 'react-native'
import Home from '../stack/home'
import Page1 from '../stack/page1'
import Page2 from '../stack/page2'
import Page3 from '../stack/page3'
import Page4 from '../stack/page4'

export const AppNavigator = createStackNavigator({
    Home: {
        screen: Home
    },
    Page1: {
        screen: Page1,
        navigationOptions: (props) => { // 动态的标题
            const { navigation } = props
            return {
                title: `${navigation.state.params.name}页面名称`
            }
        }
    },
    Page2: {
        screen: Page2,
        navigationOptions: { // 静态标题
            title: 'this is page2'
        }
    },
    Page3: {
        screen: Page3,
        navigationOptions: (props) => { // 支持在页面中修改标题
            const { navigation } = props
            const { state, setParams } = navigation
            const { params } = state
            return {
                title: params.title?params.title: 'this is page3',
                headerRight: (
                    <Button
                        title={params.mode === 'edit' ? '保存' : '编辑'}
                        onPress={() => {
                            setParams({
                                mode: params.mode==='edit'?'':'edit'
                            })
                        }}
                    />
                )
            }
        }
    },
    Page4: {
        screen: Page4,
        navigationOptions: {
            title: 'this is Page4'
        }
    }
})
```
入口js
```js
import {AppRegistry} from 'react-native';
// 路由
import { AppNavigator } from './navigators/appNavigators';
import { createAppContainer } from 'react-navigation'

import {name as appName} from './app.json';
// 包装后才能用
const navigator = createAppContainer(AppNavigator)

AppRegistry.registerComponent(appName, () => navigator);
```

### 相关的常用api
* 每个组件都会在props里面添加navigation属性
```js
this.props.navigation = {
    navigate: 跳转
    goBack: 返回
    state: 当前状态
    setParams: 对路由的参数进行更改
    getParams: 获取参数
    dispatch: 香炉有发送action
}
```
[具体详情api点击这里](https://reactnavigation.org/docs/zh-Hans/navigation-prop.html)