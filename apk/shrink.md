##### 减小apk包大小的方式：
  1. 安装包监控（每日监控，版本监控）
  2. 删除无用资源（xml，png，id，string）
  ```
  android {
    ...
    buildTypes {
      release {
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
      }
    }
  }
  ```
  3. 图片处理
    1. 对于体积特别大(超过50k)的图片资源可以考虑有损压缩，jpg采用优图压缩，png尝试采用pngquant压缩，输出视觉判断是否可行；
    2. 对assets中的图片资源也使用aapt的crunch做图片预处理；
    3. 对于没有透明区域的png图片，可以转成jpg格式。

  4. 字符串编码
    1. 从Android 2.2 API Level8开始APK文件的资源resources.arsc的编码有了小幅的改变，过去使用的是UTF-16编码方式被转换成了UTF-8编码。（纯英文的变小，中文变大）
    ```
    bool getUTF16StringsOption() {
      return mWantUTF16 || !isMinSdkAtLeast(SDK_FROYO);
    }
    ```

  5. 指定文件的压缩方式与7zip压缩
    1. 其中Method表示压缩形式，有：Deflate及Stored两种
      - Stored： 表示存储，并不压缩
      - Deflate：表示需要压缩，AssetManager读取时需要解压
    2. resources.arsc(小于1M)如果被压缩，读取时耗费内存性能
    3. 所以：那么说一般来说只有.jpg、.jpeg、.png、.gif可以压缩，对于不需要兼容低版本或者resources.arsc小于1M的apk，可选择压缩resources.arsc。



######## 瘦身记录
1. 原始包 21.2M
2. shrinkResources true   : 21M
