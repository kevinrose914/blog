## react-native离线缓存框架设计

### 3种设计模式
> 1. 优先使用本地数据，如果本地数据过期或者不存在，则从服务器获取，获取后再存储到本地
> 2. 优先使用服务器数据，从服务器获取后存储到本地，当网络出现故障时，获取本地数据
> 3. 同时从服务器和本地读取数据，如果本地数据库返回数据则先展示本地数据，等网络数据回来后，再将数据同步到本地数据，并展示

### 数据存储
```js
saveData(url, data, callback) {
    AsyncStorage.setItem(url, JSON.stringify(this._wrapData(data)), callback)
}
// 将data包装一下，增加时间戳
// 注意：这个时间戳时客户端的时间，有可能用户手动修改手机时间。故最好是从服务器取时间戳
_wrapData(data) {
    return {
        data,
        timestamp: new Date().getTime()
    }
}
```

### 获取本地数据
```js
fetchLocalData(url) {
    return new Promise((resolve, reject) => {
        AsyncStorage.getItem(url, (error, result) => {
            if (!error) {
                try {
                    resolve(JSON.parse(result))
                } catch (e) {
                    reject(e)
                    console.error(e)
                }
            } else {
                reject(error)
                console.error(error)
            }
        })
    })
}
```

### 获取网络数据
```js
fetchNetData(url) {
    return new Promise((resolve, reject) => {
        fetch(url).then(res => {
            if (res.ok) {
                return res.json()
            }
            throw new Error(Network response was not ok.)
        }).then(res => {
            // 保存到本地
            this.saveData(url, res)
            resolve(res)
        }).catch(err => {
            reject(err)
        })
    })
}
```

### 根据第一种设计模式，实现的数据获取
```js
fetchData(url) {
    return new Promise((resolve, reject) => {
        this.fetchLocalData(url).then(wrapData => {
            if (wrapData && DataStore.checkTimestampValid(wrapData.timestamp)) {
                resolve(wrapData)
            } else {
                this.fetchNetData(url).then(data => {
                    resolve(this._wrapData(data))
                }).catch(error => {
                    reject(error)
                })
            }
        }).catch(err => {
            this.fetchNetData(url).then(data => {
                resolve(this._wrapData(data))
            }).catch(error => {
                reject(error)
            })
        })
    })
}

// 检查timestamp是否在有效期内
static checkTimestampValid(timestamp) {
    // 这里是不能超过4个小时
    const currentDate = new Date()
    const targetDate = new Date()
    targetDate.setTime(timestamp)
    if (currentDate.getMonth() !== targetDate.getMonth()) {
        return false
    }
    if (currentDate.getDate() !== targetDate.getDate()) {
        return false
    }
    if ((currentDate.getHours() - targetDate.getHours()) > 4) {
        return false
    }
    return true
}
```