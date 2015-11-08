# 1 listview常用用法

listview作为展现数据的重要组件，应该被重视；我尝试着自己循环出来一个列表，发现在性能上远不及listview本身，看了api之后，发现在两个方面至少做了很好的优化

1）指定首次初始化渲染的条数

2）每次事件循环渲染的条数

这样保证在合适的时间做合适的事


# 1.1 listview

1）作为重复的列表展示

![](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/category.png)

2）按照某个属性分组展示，如 按照‘今日新闻’，‘昨日新闻’展示

![](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/sticky.png)

3）指定header和footer

4）滑动到底部时，提供刷新数据的事件入口`onEndReached`

# 1.2 listview目前没做好的事

1） 可以下拉刷新，不能上拉刷新，需要自己封装
> 我尝试封装了下，简单实现了了个demo，另外发文

# 1.3 listview 的api

## 重要属性

1）initialListSize: 初始化渲染的条数，一般大于首屏条数

2）pageSize:每次渲染的条数

3）renderScrollComponent: 返回的组件用来渲染list rows，没用过

4）scrollRenderAheadDistance: 顶部屏幕之外的数据离屏幕头部多远时开始渲染数据

5）onEndReachedThreshold: 距离底部多少时出发`onEndReached`

6）***dataSource***：接受的数据源，`ListViewDataSource`的实例对象,***必须***

7）renderSeparator：每行之间的分割线，函数类型

8）renderRow：每行的渲染函数

9）renderFooter：尾部渲染函数

10）renderHeader：头部渲染函数

11）renderSectionHeader：分组头的渲染函数

12）onChangeVisibleRows：可视化区域变化时出发，屏幕旋转时会调用

13）removeClippedSubviews： 实验性质的属性，提高滚动时到性能，和row的样式‘overflow:hidden’一起使用，没用过

## 1.4 重要方法

通过引用当前listview的实例对象可以调用的方法

1）getMetrics：返回一些常态数据


    getMetrics: function() {
      return {
        contentLength: this.scrollProperties.contentLength,
        totalRows: this.props.dataSource.getRowCount(),
        renderedRows: this.state.curRenderedRowsCount,
        visibleRows: Object.keys(this._visibleRows).length,
      };
    },
    
2） getScrollResponder：返回事件相应器对象

3） setNativeProps：动态设置样式

## 2 应用一
> 正常的列表数据，没有上拉刷新，也没有下拉刷新（其实下拉刷新就是把新拿到的数据重新重置到数据源中）

例子是1.1 中所示 ，
![](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/listview-1.png)

该例子可参考 [电影选段，啊呀呀呀呀](http://facebook.github.io/react-native/docs/tutorial.html#content),

### step 1 设置数据源

设置数据源，主要设置当前数据变化的标准，

     getInitialState: function() {
         return {
             dataSource: new ListView.DataSource({
             rowHasChanged: (row1, row2) => row1 !== row2,
         }),
         loaded: false,
     };
     
### step 2  指定渲染列

    render: function() {
        if (!this.state.loaded) {
          return this.renderLoadingView();
     }

     return (
        <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderMovie}
        style={styles.listView}
      />
     );
    },
    
### step 3 指定没行到的模版

    renderMovie: function(movie) {
        return (
            <View style={styles.container}>
            <Image
            source={{uri: movie.posters.thumbnail}}
            style={styles.thumbnail}
            />
            <View style={styles.rightContainer}>
            <Text style={styles.title}>{movie.title}</Text>
            <Text style={styles.year}>{movie.year}</Text>
            </View>
        </View>
        );
    },

### step4 样式及loading态

这是最基本的例子，具体可见 ，效果图如下

![](http://facebook.github.io/react-native/img/TutorialFinal2.png)

## 3 应用二

该例子，结合了viewpager以及listview实现左右tab滑动，上面肩头的效果用到了`animated`,listview滚动到底部可以刷新数据的功能，见下图

![](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/category_demo.gif) 

### step 1 配置和缓存
1） 设置数据类型，因为有下拉刷新的数据，当有新数据进来的时候，也只能通过***将老数据和新数据一起重新塞到*** `dataSource`中，react本身会根据`rowHasChanged`来确定是否重绘，这里有四个分类，如下

	var CACHE_DATA = {
	};
	
	var videoTypies=[
		{name:'电影',value:'movie'},
		{name:'电视剧',value:'tv'},
		{name:'综艺',value:'zy'},
		{name:'动漫',value:'katong'}
	];
	
	videoTypies.map((video) =>
		CACHE_DATA[video.value] = {
		loading: false, //当前的加载状态
		data: [], //有新数据时统统放到这，再clone到对应数据源
		pageindex: 1, //当前类型的页码
		totalcount: 0, //当前类型的数据总数
		pagecount:100//当前数据的页数
	});

由于新数据到来是由下拉动作触发的，并不是由于pageindex变化引起的，当滑动到底部时出发数据，重新加载数据，所以上面的pageindex等参数放到外面作为下次数据的参数


### step 2 初始化

初始化需要的四种数据


  	getInitialState: function() {
        return {
            page: 0,
            progress: {position: 0, offset: 0},
            movie: new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2}),
            zy: new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2}),
            tv: new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2}),
            katong: new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2}),
        };
    },


这里为了取值和设置方便，将每种类型的数据分开写了。

### step3 从渲染入口，分解


   	render: function() {
        var pages = [];
        for (var i = 0; i < videoTypies.length; i++) {
            var pageStyle = {
            backgroundColor:'#fff',
            alignItems: 'center',
            padding: 20,
        };
        var videoType = videoTypies[i];
        pages.push(
            <View key={i} style={styles.pageStyle} collapsable={true}>
            <ListView
                ref="listview"
                dataSource={this.state[videoType.value]}
                renderRow={this.renderRow}
                onEndReached={()=> { this.onEndReached(videoType.value)}}
                automaticallyAdjustContentInsets={false}
                keyboardDismissMode="on-drag"
                keyboardShouldPersistTaps={true}
                showsVerticalScrollIndicator={false}
                initialListSize={6}
                onEndReachedThreshold={50}
                pageSize={4}
                renderHeader={this.renderHeader}
                renderFooter={this.renderFooter}
            />
            </View>
        );
        }
        var page = this.state.page;
        return (
            <View style={styles.container}>
                <CategoryTabBar  go={this.go} page ={this.state.page} progress={this.state.progress}/>
                <ViewPagerAndroid
                style={styles.viewPager}
                initialPage={0}
                onPageScroll={this.onPageScroll}
                onPageSelected={this.onPageSelected}
                ref={viewPager => { this.viewPager = viewPager; }}>
                {pages}
                </ViewPagerAndroid>
            </View>
        );
    },

上面的render作为渲染的入口，这里主要是用到ViewPagerAndroid和ListView，作为主要的视窗。

1）左右滑动效果

ViewPagerAndroid实现了onPageScroll接口，返回当前event对应的到起始位置以及偏移量，这里都是按爪单位比例来算的，

 	 onPageScroll: function(e) {
        this.setState({progress: e.nativeEvent});
    },
    
 e.nativeEvent返回这样一个对象 ｛position，offset｝，
 
 position：当前移动到第几个单位，取值时0 - n，ntab的总数
 
 offset：当前的偏移量所占百分比，
 
 比如移动到第一个个和第二个之间时，position为1，offset值为 0.5，
 
 所以通过这两个值可以计算出当前视图的left值，实现滑动的效果
 
 
2）listview底部时加载下一页数据

这个实现主要通过listview本身的`onEndReached`，滚动到底部没有数据时会触发该函数，此时，会根据当前view的类型（电影还是综艺） ＋ 当前数据的页码，去获取下一页的数据，获取到下一页数据后，先放到缓存后，再统一clone到datasource中，实现翻页刷新的需求


    fetchData () {
        var page  = this.state.page;
        var type = videoTypies[page].value;
        var cache_data  = CACHE_DATA[type];
        var params = [];
        params.push('pagesize=' + PAGE_SIZE);
        params.push('type=' + type);
        //fetch data
        if (cache_data['loading']) return ;// is loading return false
        if (cache_data['pageindex'] > 0 && cache_data['pageindex'] > cache_data['pagecount']) {
            ToastAndroid.show('已经加载到最底部拉',ToastAndroid.SHORT);
            return false;
        }
        cache_data.loading = true;
        params.push('pageindex=' + cache_data.pageindex);

        var url = 'http://doukantv.com/api/top/?' + params.join('&');
        fetch(url)
        .then((res) => res.json())
        .then((res) =>{
            var list = res.result;
            var length = list.length;
            var pageinfo = res.page;
            cache_data.pagecount = pageinfo.pagecount;
            cache_data.totalcount = pageinfo.totalcount;
            cache_data.loading = false;
            list.map((video) => cache_data['data'].push(video));
            var temp = {};
            temp[type] = this.getDataSource(type, cache_data['data'])
            this.setState(temp);
        })
        .catch((error)=> {
            ToastAndroid.show('error ' + error,ToastAndroid.SHORT);
        })
    },

### step 4 其他设置


当顶部的tab切换时能够调用的回调函数告知主页切换类型了，通过`onPageSelected`实现，获取数据时用了下`this.setTimeout`，避免滑动时加载数据出现卡顿现象

    onPageSelected: function(e) {
        this.setState({page: e.nativeEvent.position});
        var page  = e.nativeEvent.position;
        var type = videoTypies[page].value;
        var cache_data  = CACHE_DATA[type];
        if (cache_data.pageindex === 1) {
            this.setTimeout(this.fetchData,200);
        }
    },


以上可参考[CategoryList.js](https://github.com/changfuguo/doukanmv/blob/master/app/android/views/CategoryList.js)

### step 5 优化

滑动的时候设置每次渲染都条数，距离到底部和顶部多远时重新绘制，这些根据自己青款设置，还有就是左右滑动的时候，请求数据可以设置一个延迟，使得用户感觉过渡会比较平滑

        initialListSize={6}
        onEndReachedThreshold={50}
        pageSize={4}

#4 应用三

上面演示了无限滚动刷新数据的功能，实际上还有一个重要功能时当前数据是按照一定标签来分类的，而listview提供了这样的功能，该功能设置和正常的设置有些不同。下面的例子不用无线滚动刷新而用固定数据来实现，先看demo

![](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/home.gif) 

### step 1 数据源

数据源的设置和之前的不一样，因为增加了分组的数据，如下面的数据

	   {
    		"result": [
       		 {
            		"list": [
               		 {
                 		 "craw": "李易峰/张慧雯/尼坤/李玟",
                		 "episode": null,
               		     "vthumburl: "http://p8.qhimg.com/t01103fa1edea6e1e05.jpg",
               		     "programname": "栀子花开2015",
               		     "shortdesc": "李易峰变国民男友",
              		      "id": 106241
              		  },
                
            		],
            		"name": "热点"
       		 }
   		  ]
          }



result里有一个分组，分组名称是“热点”，该分组对应的数据是list，这里要讲分组的数据按照如下形式进行拼装，
![](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/listview-2.jpg) 

初始化数据如下


					getInitialState: function() {
                        var getSectionData = (dataBlob, sectionID) => {
                            return dataBlob[sectionID];
                        };

                        var getRowData = (dataBlob, sectionID, rowID) => {
                            return dataBlob[sectionID + ':' + rowID];
                        };

                        var dataSource = function () {
                            return new ListView.DataSource({
                               getRowData: getRowData,
                               getSectionHeaderData: getSectionData,
                               rowHasChanged: (row1, row2) => row1 !== row2,
                               sectionHeaderHasChanged: (s1, s2) => s1 !== s2
                            });
                        }
                        return {
                            page: 0,
                            progress: {position: 0, offset: 0},
                            waitingForRelease: false,
                            isRefreshing: false,
                            movie: {
                                data: dataSource(),
                                loading : 0
                            },
                            zy : {
                                data: dataSource(),
                                loading : 0
                            },
                            tv: {
                                data: dataSource(),
                                loading : 0
                            },
                            katong: {
                                data: dataSource(),
                                loading : 0

                            }
                        };
               },

设置数据源时需要和之前的不一样，需要设置`sectionHeaderHasChanged`,当数据源中section信息改变时根据设置的`sectionHeaderHasChanged`作为是否重新绘制的依据

### step 2 渲染入口

除了在listview中增加一个`renderSectionHeader={this.renderSectionHeader}`之外，其余和其他的是一样的

`this.renderSectionHeader`函数如下

                renderSectionHeader: function(sectionData: Object,
                    sectionID: number | string) {
                        return (
                            <View style={styles.sectionHeader}>
                                <Text >| {sectionData.name}</Text>
                            </View>
                    );

                },
           
### step 3 数据渲染

获取数据如下，获取到数据后进行变形，

					fetchData () {
                        var page  = this.state.page;
                        var type = videoTypies[page].value;

                        var loading = this.state[type].loading;
                        if(loading === 1) return;
                        var url = 'http://doukantv.com/api/hot/?type=' + type;
                        this.state[type].loading = 1;
                        fetch(url)
                            .then((res) => res.json())
                            .then((res) =>{
                                var temp ={};
                                temp[type] = {
                                    data: this.getDataSource(type, res.result),
                                    loading: 0
                                }
                                this.setState(temp);
                            })
                    },


数据变形

                getDataSource: function(type, videos): ListView.DataSource {
                    var data  = this.state[type].data;

                    var dataBlob = {};
                    var sectionsID = [];
                    var rowsID = [];
                    videos.map((item, index)=>{
                        sectionsID.push(index);
                        dataBlob[index] = {name: item.name};
                        var rowID = rowsID[index] = [];
                        item.list.map((video) => {
                            var id = video.id;
                            dataBlob[index + ':' + id] = video;
                            rowID.push(id);
                        });
                    });
                    return data.cloneWithRowsAndSections(dataBlob, sectionsID, rowsID);
                },
                
                
 经过变形后的数据格式可以直接传入到listview的datasource中了，代码可以参考[hot.js](https://github.com/changfuguo/doukanmv/blob/master/app/android/views/Hot.js)
 
 
 
# 5 遗留问题


listview作为数据展示的主要控件，作用不言而喻，但是facebook官方没有提供下拉到顶部时刷新的入口，咨询了native端的同学，说在native端也是需要自己写插件，详细可见另外的文章


