# ReactNative Misc

(勉强把这个归类到 frontend 类别下。)

记录 ReactNative 开发中遇到的一些问题及解决办法。

## Image 全屏的问题

这个问题前前后后遇到了三次，之前没有记录下来，在第三次遇到的时候就又抓狂了。

这个问题出现在 android 上，iOS 上没有问题。在 iOS 上给 image 设置属性 `resizeMode='stretch'` 和 style `{ flex: 1 }` 就够了，但是在 android 上，还必须要给 style 加上 `{ width: null, height: null }` 才 work，原因未知。

完整的示例代码：

    <Image source={require(...)}
           style={styles.imgBg}
           resizeMode='stretch'>
    </Image>

    imgBg: {
      flex: 1,
      width: null,
      height: null,
    }
