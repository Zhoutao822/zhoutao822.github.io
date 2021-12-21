---
title: "Android Lint"
date: 2021-12-13T21:31:53+08:00
tags: [""]
categories: [""]
series: [""]
summary: "Summary todo"
draft: true
mathjax: true
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

## 1. 常见Lint error

1. AutoLink: android:autoLink not allow in TextView
2. DebugToLogEnable: Should not use BuildConfig.DEBUG
3. GradientScan: <path> fillColor/strokeColor use <gradient> only avaliable on api24
4. L10NScan: Check invalid usage of l10n strings
5. LogToZMLog: Should not use android.util.Log
6. LogWithinDebug: Should not use ZMLog outside if(BuildConfig.LOG_ENABLE)
7. RawStringTest: Raw string contains "test" is not allowed
8. StartActivity: Should not use startActivity directly
9. OldTargetApi: Target SDK attribute is not targeting latest version
10. EllipsizeMaxLines: Combining Ellipsize and Maxlines
11. StringFormatInvalid: Invalid format string
12. StringFormatMatches: `String.format` string doesn't match the XML format string
13. PluralsCandidate: Potential Plurals
14. StringFormatCount: Formatting argument types incomplete or inconsistent
15. SyntheticAccessor: Synthetic Accessor

这些lint示例可以在[lintchecks][https://git.zoom.us/main/android_cn/-/tree/master/lintchecks]项目中找到，仓库位于https://git.zoom.us/main/android_cn，其中包含一些自定义的lint规则用于代码规范

### 1.1 AutoLink

不要在TextView属性中使用**android:autoLink**

```xml
android:layout_height="wrap_content"
android:text="@string/hello"
android:textAllCaps="true"
android:autoLink="all"
                                                                     
app:layout_constraintBottom_toBottomOf="parent"
app:layout_constraintLeft_toLeftOf="parent"
app:layout_constraintRight_toRightOf="parent"
```

### 1.2 DebugToLogEnable

不要使用BuildConfig.DEBUG，用**BuildConfig.LOG_ENABLE**替代

```java
if (BuildConfig.DEBUG){    

}
```

### 1.3 GradientScan

<gradient> 在api24以下可能无法解析导致crash，如果资源文件中有这样的用法需要同时提供用在api21设备上的同名资源文件（文件中没有使用到<gradient>）

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:aapt="http://schemas.android.com/aapt"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
  <path
      android:pathData="M6.4806,8.3938C4.4996,8.3938 2.8937,9.9997 2.8937,11.9807C2.8937,13.9617 4.4996,15.5676 6.4806,15.5676C7.399,15.5676 8.2359,15.2233 8.871,14.655V13.1798H6.7797C6.2827,13.1798 5.8797,12.7768 5.8797,12.2798C5.8797,11.7827 6.2827,11.3798 6.7797,11.3798H9.771C10.2681,11.3798 10.671,11.7827 10.671,12.2798V15.0312C10.671,15.2582 10.5852,15.4768 10.4309,15.6432C9.4482,16.7026 8.0414,17.3676 6.4806,17.3676C3.5055,17.3676 1.0937,14.9558 1.0937,11.9807C1.0937,9.0056 3.5055,6.5938 6.4806,6.5938C7.6923,6.5938 8.8129,6.9949 9.7133,7.6712C10.1107,7.9697 10.1909,8.5339 9.8924,8.9313C9.5939,9.3287 9.0297,9.4089 8.6322,9.1104C8.0329,8.6602 7.2892,8.3938 6.4806,8.3938ZM15.722,7.4952C15.722,6.9981 16.1249,6.5952 16.622,6.5952H22.0063C22.5033,6.5952 22.9063,6.9981 22.9063,7.4952C22.9063,7.9923 22.5033,8.3952 22.0063,8.3952H17.522V11.3812H22.0063C22.5033,11.3812 22.9063,11.7842 22.9063,12.2812C22.9063,12.7783 22.5033,13.1812 22.0063,13.1812H17.522V16.469C17.522,16.9661 17.119,17.369 16.622,17.369C16.1249,17.369 15.722,16.9661 15.722,16.469V12.2812V7.4952ZM14.0954,7.4952C14.0954,6.9981 13.6924,6.5952 13.1954,6.5952C12.6983,6.5952 12.2954,6.9981 12.2954,7.4952V16.469C12.2954,16.9661 12.6983,17.369 13.1954,17.369C13.6924,17.369 14.0954,16.9661 14.0954,16.469V7.4952Z"
      android:fillType="evenOdd">
    <aapt:attr name="android:fillColor">
      <gradient 
          android:startY="17.4057"
          android:startX="22.9063"
          android:endY="5.94725"
          android:endX="1.17547"
          android:type="linear">
        <item android:offset="0" android:color="#D04F3E"/>
        <item android:offset="1" android:color="#5381E2"/>
      </gradient>
    </aapt:attr>
  </path>
</vector>

```

### 1.4 L10NScan

strings.xml中利用正则式匹配`</string>.+|</plurals>.+|</item>.+`，如果匹配成功说明其中存在问题字符串

### 1.5 LogToZMLog

不要使用Log类，请使用**ZMLog**

```java
Log.i("", "");                                                                              
Log.d("", "");
Log.e("", "");
Log.v("", "");
```

### 1.6 LogWithinDebug

使用ZMLog时，请将其放在**BuildConfig.LOG_ENABLE**条件中

```java
if (BuildConfig.DEBUG) {
	ZMLog.d("kotlin1", "")
}
```

### 1.7 RawStringTest

不要直接使用`test`字符串

```java
String a = "test";
```

### 1.8 StartActivity

不要直接使用startActivity方法，请使用**ActivityStartHelper.startActivity**

```kotlin
class KotlinActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        startActivity(Intent());
    }
}
```

## 2. 自定义lint

首先需要clone https://git.zoom.us/main/android_cn仓库，代码在[lintchecks][https://git.zoom.us/main/android_cn/-/tree/master/lintchecks]项目中，lintchecks工程中有3个module: `checks`, `library`, `sample`，其中`library`不用管，`sample`用于编写用来测试lint功能的代码，`checks`用于编写自定义lint，以`DebugAndLogDetector`为例

```java
public class DebugAndLogDetector extends Detector implements SourceCodeScanner {
```

这个DebugAndLogDetector用于扫描`DebugToLogEnable`, `LogToZMLog`, `StartActivity`, `LogWithinDebug`4个规则，这4个规则都是扫描java代码或者kotlin代码，因此直接继承Detector并实现SourceCodeScanner接口即可，如果需要扫描的是xml资源文件，则继承ResourceXmlDetector，具体可参考SVGDetector。

然后需要定义一个Issue，用于表示此lint规则的相关信息，比如危险等级、概要、解释、id、优先级等

```java
public static final Issue ISSUE_DEBUG = Issue.create(
        "DebugToLogEnable",
        "Should not use BuildConfig.DEBUG",
        "change \"BuildConfig.DEBUG\" to \"BuildConfig.LOG_ENABLE\"",
        Category.CORRECTNESS,
        8,
        Severity.ERROR, IMPLEMENTATION)
        .setAndroidSpecific(true);
```

Implementation用于定义scope以及绑定Issue，一般都是默认写法，扫描java代码使用Scope.JAVA_FILE_SCOPE，扫描xml使用Scope.RESOURCE_FILE_SCOPE

```java
private static final Implementation IMPLEMENTATION =
    new Implementation(DebugAndLogDetector.class, Scope.JAVA_FILE_SCOPE);
```




## 参考

1. [自定义 Lint 检查实践指南](https://zhuanlan.zhihu.com/p/307382854)
2. [Android自定义Lint实践](https://tech.meituan.com/2016/03/21/android-custom-lint.html)
3. [android自定义Lint实现代码检测](https://juejin.cn/post/6844903774104879111)
4. [googlesamples/android-custom-lint-rules](https://github.com/googlesamples/android-custom-lint-rules)
4. [Implementing a Parser and PSI](https://kana112233.github.io/intellij-sdk-docs-cn/reference_guide/custom_language_support/implementing_parser_and_psi.html)

