## 强调

*加倾斜效果*   
**加粗**   
***加粗、加斜线***    
_加下划线_   
_***加粗、加倾斜、加下划线***_   
~~删除线~~    
添加脚注:[^1]    

[^1]:这里是脚注内容


## 列表
* 无序列表
* 无序列表二
	* 二级无序列表一
		* 三级无序列表

1. 一级有序列表一 `数字+空格+.`
	2. 二级有序列
		3. 三级有序
4. 二级无序列表二


- [ ] 任务列表 `- + 空格 + [空格]`
- [x] 任务列表

## 表格

第一列表头|第二列表头
---------|--------
第二行1列| 第二行2列
---------|--------


## 图片
格式 ![]()
快捷键 `shift + command + i`
   ![百度图片](http://zh.mweb.im/asset/img/set-up-git.gif)

## 链接
Email自动生成，格式`<**@**.com>`：<1433495875@qq.com>
网站链接格式：[]()[网站链接标题](http://www.baidu/com),快捷键`control + shift + l`
自动生成链接，格式`<http://******`<http://www.baidu.com>

## 分割线
***
*****
- - -

## 区域模块
> 区域模块第一行
> 区域模块第二行
	> 缩进

## 代码模块
```java
/** load url with given referer or default referer */
public static void loadWebViewUrl(String url, WebView webview, String referer, boolean setDefaultReferer) {
    if (webview == null || StringUtils.isEmpty(url)) {
        return;
    }
    boolean isHttp = isHttpUrl(url);
    if (isHttp && setDefaultReferer && StringUtils.isEmpty(referer)) {
        referer = BaseAppData.inst().getHttpReferer();
    }
    if (!isHttp) {
        referer = null;
    }
    HashMap<String, String> headers = new HashMap<String, String>();
    if (!StringUtils.isEmpty(referer)) {
        headers.put(AbsConstants.HTTP_H_REFERER, referer);
    }
    loadWebViewUrl(url, webview, headers);
}

```
## 顺序或者流程图
`目前无法支持`
```sequence
张三->李四: 嘿，小四儿, 写博客了没?
Note right of 李四: 李四愣了一下，说：
李四-->张三: 忙得吐血，哪有时间写。
```

```flow
st=>start: 开始
e=>end: 结束
op=>operation: 我的操作
cond=>condition: 确认？

st->op->cond
cond(yes)->e
cond(no)->op
```

##



的
