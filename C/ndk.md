ndk##### Android.mk 文件

  ```
  LOCAL_PATH := $(call my-dir)                            指定Android.mk所在路径
  include $(CLEAR_VARS)                                   清空local变量
  LOCAL_MODULE    := daemon_api20                         指定Android.mk每一个模块的名称
  LOCAL_SRC_FILES := daemon_api20.c \                     将要被打包的c源码
  	common.c
  LOCAL_C_INCLUDES := $(LOCAL_PATH)/include               加载NDK ROOT的某个库，编译时会携带上
  LOCAL_LDLIBS := -L$(SYSROOT)/usr/lib -llog -lm -lz      添加系统库
  include $(BUILD_SHARED_LIBRARY)                         指向一个GNU Makefile Script。收集上次调CLEAR_VARS之后的本地变量


  include $(CLEAR_VARS)                                   清空local变量
  LOCAL_MODULE    := daemon_api21                         Android.mk模块名称
  LOCAL_SRC_FILES := daemon_api21.c \                     需要被编译的C源码
  	common.c
  LOCAL_C_INCLUDES := $(LOCAL_PATH)/include               需要加载的相对于NDK ROOT的库
  LOCAL_LDLIBS := -L$(SYSROOT)/usr/lib -llog -lm -lz      需要加载的系统库
  include $(BUILD_SHARED_LIBRARY)                         指向一个GUN脚本，手机local变量


  include $(CLEAR_VARS)                                   清空local变量
  LOCAL_MODULE    := daemon                               Android.mk模块名称
  LOCAL_SRC_FILES := daemon.c                             需要编译的C源码
  LOCAL_CFLAGS += -pie -fPIE                              设置本地代码的flag
  LOCAL_LDFLAGS += -pie -fPIE
  LOCAL_C_INCLUDES := $(LOCAL_PATH)/include               加载本地C代码的路径
  LOCAL_LDLIBS += -L$(SYSROOT)/usr/lib -llog -lm -lz      加载系统库
  include $(BUILD_EXECUTABLE)                             编译为Native C可执行程序  


  ```
  - LOCAL_PATH := $(call my-dir)      
    每个Android.mk文件必须以定义LOCAL_PATH为开始。它用于在开发tree中查找源文件。宏my-dir 则由Build System提供。返回包含Android.mk的目录路径。
  - include $(CLEAR_VARS)             
    CLEAR_VARS由Build System提供，做清理Local全局变量的情路，防止相互干扰
  - LOCAL_MODULE    := hello-jni      
    LOCAL_MODULE模块必须定义，以表示Android.mk中的每一个模块。名字必须唯一且不包含空格。Build System会自动添加适当的前缀和后缀。例如，foo，要产生动态库，则生成libfoo.so.
  - LOCAL_SRC_FILES := hello-jni.c    
    LOCAL_SRC_FILES变量必须包含将要打包如模块的C/C++ 源码。不必列出头文件，build System 会自动帮我们找出依赖文件。缺省的C++源码的扩展名为.cpp. 也可以修改，通过LOCAL_CPP_EXTENSION。
  - LOCAL_C_INCLUDES := $(LOCAL_PATH)/include   
    一个可选的path列表。相对于NDK ROOT 目录。编译时，将会把这些目录附上。LOCAL_C_INCLUDES := sources/foo  LOCAL_C_INCLUDES := $(LOCAL_PATH)/../foo
  - LOCAL_LDLIBS := -L$(SYSROOT)/usr/lib -llog -lm -lz
    linker flags。 可以用它来添加系统库。 如 -lz:  LOCAL_LDLIBS := -lz  
  - LOCAL_CFLAGS += -pie -fPIE：
    一个可选的设置，在编译C/C++ source 时添加如Flags。
  - LOCAL_LDFLAGS += -pie -fPIE：
    没有查到



  - include $(BUILD_SHARED_LIBRARY)   
    BUILD_SHARED_LIBRARY：是Build System提供的一个变量，指向一个GNU Makefile Script。它负责收集自从上次调用 include $(CLEAR_VARS)  后的所有LOCAL_XXX信息。并决定编译为什么。




###### NDK提供的变量：
  此类GNU Make变量是NDK Build System在解析Android.mk之前就定义好了的。
  2.1.1：CLEAR_VARS：
    指向一个编译脚本。必须在新模块前包含之。
    include $(CLEAR_VARS)

  2.1.2：BUILD_SHARED_LIBRARY：
    指向一个编译脚本，它收集自从上次调用 include $(CLEAR_VARS)  后的所有LOCAL_XXX信息。
    并决定如何将你列出的Source编译成一个动态库。 注意，在包含此文件前，至少应该包含：LOCAL_MODULE and LOCAL_SRC_FILES 例如：
    include $(BUILD_SHARED_LIBRARY)      

  2.1.3：BUILD_STATIC_LIBRARY：
    与前面类似，它也指向一个编译脚本，
    收集自从上次调用 include $(CLEAR_VARS)  后的所有LOCAL_XXX信息。
    并决定如何将你列出的Source编译成一个静态库。 静态库不能够加入到Project 或者APK中。但它可以用来生成动态库。
    LOCAL_STATIC_LIBRARIES and LOCAL_WHOLE_STATIC_LIBRARIES将描述之。    
    include $(BUILD_STATIC_LIBRARY)

  2.1.4: BUILD_EXECUTABLE:
    与前面类似，它也指向一个编译脚本，收集自从上次调用 include $(CLEAR_VARS)  后的所有LOCAL_XXX信息。
    并决定如何将你列出的Source编译成一个可执行Native程序。  
    include $(BUILD_EXECUTABLE)

  2.1.5：PREBUILT_SHARED_LIBRARY：
    把这个共享库声明为 “一个” 独立的模块。
    指向一个build 脚本，用来指定一个预先编译好多动态库。 与BUILD_SHARED_LIBRARY and BUILD_STATIC_LIBRARY不同，
    此时模块的LOCAL_SRC_FILES应该被指定为一个预先编译好的动态库，而非source file.  LOCAL_PATH := $(call my-dir)
