# react－native 之listview下拉刷新的尝试 
```redux```
react－native出来很久了，android版本也有两个月了，尝试着写了下android，可参考[doukanmv](https://github.com/changfuguo/doukanmv),写到半路发现下拉刷新暂时需要自己封装，查了下找到一个[react-native-refreshable-listview](https://github.com/jsdf/react-native-refreshable-listview)，最后更新都是几个月前的了，试了下果然不能用，于是自己想着尝试下，[代码在这里Search.js ]
(https://github.com/changfuguo/doukanmv/blob/master/app/android/views/Search.js)最后的效果如下：

![pulldown](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/pulldown.gif)

下拉滚动的逻辑如下（图中‘是否正在刷新’ 处有误，n和y的位置反了）：

![pulldown](https://raw.githubusercontent.com/changfuguo/doukanmv/master/temp/logic_for_pulldown.jpg)


# 1、实验一：listview ＋ 绝对定位

listview设置成绝对定位，通过本身的onScroll事件监听当前运动的位置，在listview上面放置一个view作为下拉提示的视图，但是刚刚设置完样式后，发现将listview放置到view中时,下拉没有响应任何事件，于是失败，其结构如下：

	return (

        <View key='1'>
            {this.renderIndicator()}
        <ListView
          ref={listview => { this.listview = listview; }}
          dataSource={this.state.dataSource}
          renderRow={this._renderRow}
          automaticallyAdjustContentInsets={false}
          keyboardDismissMode="on-drag"
          scrollRenderAheadDistance= {100}
          keyboardShouldPersistTaps={false}
          showsVerticalScrollIndicator={true}
          initialListSize={12}
          onEndReachedThreshold={50}
          pageSize={6}
          style={[styles.list,listStyle]}
          />
    </View>

    );
    
# 2、实验二：scrollview＋listview

***上述实验时，用绝对定位的top实现listview向下自动滑动效果，发现绝对定位有问题，整个listview内容不显示了，暂时没追查原因，换了个思路。查了下stylesheet文档发现支持translateY,所以这里采用translateY实现下移效果***

1、判定拉倒顶部

scrollview 有onScroll事件，可以探测是否滑正在从列表顶部往下来：

onScroll事件传入的事件类型中

  	listViewScroll (e) {
    	this.scrollY =  e.nativeEvent.contentOffset.y;
  	},
  	
  	
当拉倒顶部之前e.nativeEvent.contentOffset.y返回一个大于0的值，拉倒顶部时返回值为0，可以根据这个值来判定是否拉倒顶部
  	
 2、怕断是否正在触屏
 
 这里用PanResponder触发的触屏事件和方法，具体可见[panresponder](http://facebook.github.io/react-native/docs/panresponder.html#content)
 
 
 我们关心几个方法
 
 
 	onStartShouldSetPanResponder: (evt, gestureState) => {
        	return !that.state.refreshing;
      },
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,

      onPanResponderGrant: (evt, gestureState) => {
         this.setState({touching: true});
         console.log('onPanResponderGrant');
      },
     
      onPanResponderMove: (evt, gestureState) => {

         if(that.scrollY > 0 || that.state.refreshing) {
            return ;
         }

         var dy = that.dy = gestureState.dy * SCROLL_ACC;

         if(dy > MIN_PULLDOWN_DISTANCE && that.state.touching && !that.state.refreshing) {
              that.setState({scrollY:dy});
               that._genRows();
              that.onRefresh(dy);
               console.log(dy);
         }

         console.log('onPanResponderMove');
      },
      onPanResponderEnd: (evt, gestureState) => {
                that.onRelease();
                that.setState({touching: false});
                console.log('onPanResponderEnd');

      },
    });

 onStartShouldSetPanResponder：是否要成为相应器
 
 onMoveShouldSetPanResponder：手指移动时是否响应
 
 onPanResponderGrant：授权称为相应器
 
 onPanResponderMove：手指移动时触发
 
 onPanResponderEnd：手指起来时触发
 
 
 
 > ps： 并不是手指移动时，并非会一直触发onPanResponderMove，之后就触发onPanResponderEnd，这个是在一定时间内响应的，超过了之后就会自动释放
 
 当前scrollview授权成为响应起器之后，认为触摸开始
 
 3、判断何时刷新
 
2中touching之后向下拉动的值测试了一下一般是小于20的数，listview向下滚动的高度一般有几十pt，所以这里采用了一个线性变换，move时得到的dy值乘以SCROLL_ACC（这里取5）.
 
当滚动到顶部且数据没处于正在刷新状态，且变换后的高度小于我们设定的溢值，这里是MIN_PULLDOWN_DISTANCE，我们进入刷新数据的状态，触发请求数据的函数`that._genRows()`，即更改数据请求状态that.state.refreshing为true，同时更新listview的位置以及顶部提示信息的高度，看上去像是两个一起进行运动的。
 
	that._genRows();  //重新请求数据
    that.onRefresh(dy); //重新设定listview的translateY的值
 
更新高度逻辑如下，dy变换后的高度是不能大于我们设定的溢值SCROLL_HEIGHT（这里取150）


	onRefresh (dy) {
      this.setState({scrollY:dy > SCROLL_HEIGHT ? SCROLL_HEIGHT: dy});
	},
	
	
这里有一个问题是，当手指起来的时候，此时如果scroll的高度还没有达到我们设定的溢值SCROLL_HEIGHT，这里要讲剩下的高度补齐，通过`onPanResponderEnd`函数调用`that.onRelease`

	onRelease () {
       var dy = this.dy ;
       var hander ;
       if(this.state.refreshing && this.dy > 3 && dy < SCROLL_HEIGHT) {
              hander = setInterval(()=>{
                if(dy + 20 < SCROLL_HEIGHT){
                    dy = dy + 20;
                    this.setState({scrollY: dy});
                }else{
                    this.dy = SCROLL_HEIGHT ;
                    clearInterval(hander);
                    this.setState({scrollY:SCROLL_HEIGHT});
                }
              },30);
       }
	},
	
4、数据请求完毕，复位

数据一般都是一个异步请求，请求完成之前refreshing的值都是true，再下拉活着其他请求数据时（比如此时下拉）都不做响应；当数据完成之后，需要将translateY的高度复位，请求数据逻辑如下：



	_genRows: function(pressData: {[key: number]: boolean}): Array<string> {


    if(this.state.refreshing) return;
    var number = Math.round(Math.random()*30);
    var data = [];
    number =20;
    for(var i = 0;i < number; i++) {
    if(i == 0 || i == 1 || i == 2) {
        data.push(Math.random());
    } else {
        data.push(data[i-1] + data[i-2])
    }
    }
    this.setState({refreshing:true});
    var that = this;
    this.setTimeout(function(){
        that.setState({refreshing: false});
        that.setState({dataSource: that.state.dataSource.cloneWithRows(data.reverse())});

        var hander  = setInterval(()=>{
            if(that.state.scrollY - 20 > 0 ){
                that.setState({scrollY:that.state.scrollY-20})
            } else {
                clearInterval(hander);
                that.setState({scrollY:0})
            }

        },40);

    },3000)

	},
 
 这里定时器模拟数据请求的过程，随即生成数据，请求完成之后，减少`scrollY`，复位，可以开始下一轮请求
 
 
 
# 3 坑

这里统一用this.state.scrollY来表示滑动的距离，listview滑动到溢值的位置时，发现上面的文字不见了。

后来进来页面时就设置this.state.scrollY（translateY）为150，设置提示信息所在view的高度为50，发现正好能显示出来，（提示view设置高度为49，加上背景色，发现上图红色和绿色之间有一条白的间隙，改成50就没有了），所以也很奇怪这中间还有一个换算比例吗？所以设置提示view的高度为150时，文字居中到红色下面了。回头找个时间验证下再，反正this.state.scrollY变化时，提示view的高度设置为this.state.scrollY/3，发现正好可以覆盖。最终的效果如图1所示。


# 4 总结 

这个感觉要是native实现效果会更好，这里都算是实验，目前这个实现点效果应该能满足基本需求，各位看官自行取舍。代码放在[Search.js]()
