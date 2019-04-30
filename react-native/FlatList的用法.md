## FlatList
```js
import {FlatList, RefreshControl} from 'react-native'
<FlatList
    data={store.projectModes} // 源数据
    renderItem={data => this.renderItem(data)} // 每一项的dom
    key={({item}) => item.id} // 唯一标识
    refreshControl={ // 下拉刷新的ui
        <RefreshControl
            title={"Loading"}
            titleColor={THEME_COLOR}
            colors={[THEME_COLOR]}
            refreshing={store.isLoading} // 是否显示
            onRefresh={() => this.loadData()}
            tintColor={THEME_COLOR}
        />
    }
    ListFooterComponent={() => {// 定义上拉刷新的控件
        return this.genIndicator()
    }} 
    onEndReached={() => {
        // 这儿用一个开关，主要是滑动到底部，会调用两次这个函数，造成性能问题
        // settimeout确保该回调函数是在onMomentumScrollBegin之后执行
        setTimeout(() => {
            if (this.canLoadMore) {
                this.canLoadMore = false
                this.loadData(true)
            }
        }, 100)
    }}
    onEndReachedThreshold={0.5}
    onMomentumScrollBegin={() => {
        this.canLoadMore = true
    }}
/>
renderItem(data) {
    const { item } = data
    return <PopularItem
        item={item}
        onSelect={() => {

        }}
    />
}

genIndicator() {
    return this._store().hideLoadingMore ? null : (
        <View style={styles.indicatorC}>
            <ActivityIndicator
                style={styles.indicator}
                size={'large'}
                color={'red'}
                animating={true}
            />
            <Text>正在加载更多</Text>
        </View>
    )
}
```