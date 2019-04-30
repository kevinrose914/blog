## AsyncStorage的用法

### 存储数据
```js
import { AsyncStorage } from 'react-native'
async doSave() {
    // 用法一
    AsyncStorage.setItem(KEY, this.value, error => {
        error && console.log(error.toString())
    })
    
    // 用法二
    AsyncStorage.setItem(KEY, this.value).catch(error => {
        error && console.log(error.toString())
    })

    // 用法三
    try {
        await AsyncStorage.setItem(KEY, this.value)
    } catch (error) {
        error && console.log(error.toString())
    }
}
```

### 读取数据
```js
import { AsyncStorage } from 'react-native'
async getData() {
    // 用法一
    AsyncStorage.getItem(KEY, (error, value) => {

    })
    
    // 用法二
    AsyncStorage.getItem(KEY).then(val => {

    }).catch(error => {
        error && console.log(error.toString())
    })

    // 用法三
    try {
        const value = await AsyncStorage.getItem(KEY)
    } catch (error) {
        error && console.log(error.toString())
    }
}
```

### 删除数据
```js
import { AsyncStorage } from 'react-native'
async doRemove() {
    // 用法一
    AsyncStorage.removeItem(KEY, error => {

    })
    
    // 用法二
    AsyncStorage.removeItem(KEY).catch(error => {
        error && console.log(error.toString())
    })

    // 用法三
    try {
        await AsyncStorage.removeItem(KEY)
    } catch (error) {
        error && console.log(error.toString())
    }
}
```

### 其他api
> * mergeItem(key, value): 修改
> * clear: 清空所有
> * getALLkeys()
> * multiGet(keys:[], cb, result:[]): 多个获取
> * multiSet()
> * multiRemove()
> * multiMerge()

### 注意
> * AsyncStorage在将来会被移出react-native，然后可以从'@react-native-community/async-storage'这里install