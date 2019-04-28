# redux配合react-navigation的使用
### 安装第三方
> npm install --save redux react-redux redux-thunk react-navigation-redux-helpers
> npm install --save-dev redux-devtools

### 配置navigation
```js
import { 
    createStackNavigator,
    createSwitchNavigator,
    createAppContainer
} from 'react-navigation'
import { createReduxContainer } = 'react-navigation-redux-helpers'
const MainNav = createStackNavigator({
    ...
})
const InitNav = createStackNavigator({
    ...
})
const RootNav = createAppContainer(createSwitchNavigator({
    Main: MainNav,
    Init: InitNav
}))
// 包装RootNav
const AppNavigator = createReduxContainer(RootNav)
const mapStateToProps = state => ({
    state: state.nav
})
// 暴露App
export default connect(mapStateToProps)(AppNavigator)
```
### 配置导航的reducer
```js
// 第一种方式：使用react-navigation-redux-helpers自带的
import { createNavigationReducer } from 'react-navigation-redux-helpers'
// AppNavigator为上面的RootNav
const navReducer = createNavigationReducer(AppNavigator)

// 第二种方式：自己配置
// 指定默认state, RootNavigator为上面的RootNav
const rootCom = 'Init' // 定义根路由
const action = RootNavigator.router.getActionForPathAndParams(rootCom)
const navState = RootNavigator.router.getStateForAction(action)
// 创建自己的navigation reducer
const navReducer = (state = navState, action) => {
    const nextState = RootNavigator.router.getStateForAction(action, state)
    return nextState || state
}
const reducers = combineReducers({
    nav: navReducer,
    ...
})
```

### 配置导航中间件
```js
import { createReactNavigationReduxMiddleware } = 'react-navigation-redux-helpers';
const middleware = createReactNavigationReduxMiddleware(
    // 'root', 这儿这个root，写的时候会报错，暂时不知是为何
    state => state.nav
)
const middlewares = [middleware]
const store = createStore(reducers, applyMiddleware(...middlewares))
```

### app.js
```js
// AppNavigator为第一个js中最后暴露出来的
import { Provider } from 'react-redux'
class App extends Component {
    render() {
        return (
            <Provider store={store}>
                <AppNavigator />
            </Provider>
        )
    }
}
```

### 入口文件
```js
import {AppRegistry} from 'react-native';
import App from './js/App'; // 为上面的App
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

### 处理安卓设备物理返回键的问题
```js
// home.js
class HomePage extends Component<Props> {
    constructor(props) {
        super(props)
        this.onBackPress = this.onBackPress.bind(this)
    }
    render() {
        return (
            <DynamicTabNavigator { ...this.props }/>
        )
    }
    /**
     * 处理安卓设备物理返回键的问题
     */
    componentDidMount() {
        BackHandler.addEventListener('hardwareBackPress', this.onBackPress)
    }
    componentWillUnmount() {
        BackHandler.removeEventListener('hardwareBackPress', this.onBackPress)
    }
    onBackPress() {
        const { dispatch, nav } = this.props
        if (nav.routes[1].index === 0) { // nav.routes[1]为Main
            return false
        }
        dispatch(NavigationActions.back())
        return true
    }
}
const mapStateToProps = state => {
    return {
        nav: state.nav
    }
}
export default connect(mapStateToProps)(HomePage)
```