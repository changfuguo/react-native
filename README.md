> 这里记录我在react-native开发过程中遇到的各种问题及相关资源汇总

# 1、资料汇总 

1   官方文档必备：[http://facebook.github.io/react-native/docs/getting-started.html#content](官方文档)


2   maven 国内镜像 及gradle的配置方法
  天朝下载打包依赖实在是慢的要死，改成国内的 刷刷的，再也不用为翻墙发愁了
  [Gradle 修改 Maven 仓库地址](http://www.yrom.net/blog/2015/02/07/change-gradle-maven-repo-url/)
  
----------
  一个简单的办法，修改项目根目录下的build.gradle，将jcenter()或者mavenCentral()替换掉即可：

  allprojects {
      repositories {
          maven{ url 'http://maven.oschina.net/content/groups/public/'}
      }
  }
  
# 2常用组件 

1、顶部是导航栏 下面的当行的内容  
    [react-native-scrollable-tab-view](https://github.com/brentvatne/react-native-scrollable-tab-view)
    看图
    ![](https://raw.githubusercontent.com/brentvatne/react-native-scrollable-tab-view/master/demo.gif)
    ![](https://raw.githubusercontent.com/brentvatne/react-native-scrollable-tab-view/master/demo-fb.gif)
