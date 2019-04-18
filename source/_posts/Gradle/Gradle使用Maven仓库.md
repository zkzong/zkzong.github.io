1. 找到maven仓库的位置并复制路径`E:\repository`。
![maven仓库地址](https://upload-images.jianshu.io/upload_images/292448-0499af45d4755a24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 配置环境变量`GRADLE_USER_HOME`，将上面的`E:\repository`配置进去。
![GRADLE_USER_HOME](https://upload-images.jianshu.io/upload_images/292448-527c40a6998faee2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 重启idea验证是否生效。可以看到Gradle模块的`Service directory path`变成了我们设置的`GRADLE_USER_HOME`环境变量的位置，说明gradle用maven的仓库生效了。
![Gradle路径](https://upload-images.jianshu.io/upload_images/292448-5a0cbca8f75c5155.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 配置gradle工程文件加载jar包仓库的优先级，让其先从本地仓库找，如果找不到，再从中央仓库下载。
```
repositories {
    /**
     * 先让gradle从本地仓库找,找不到再从下面的mavenCentral()中央仓库去找jar包
     */
    mavenLocal()
    mavenCentral()
}
```
![mavenLocal](https://upload-images.jianshu.io/upload_images/292448-4077d3f60f28b985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
