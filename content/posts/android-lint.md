---
title: "Android Lint"
date: 2021-12-13T21:31:53+08:00
tags: ["lint", "android"]
categories: ["android"]
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

1. [AutoLink](#AutoLink): android:autoLink not allow in TextView
2. [DebugToLogEnable](#DebugToLogEnable): Should not use BuildConfig.DEBUG
3. [GradientScan](#GradientScan): <path> fillColor/strokeColor use <gradient> only avaliable on api24
4. [L10NScan](#L10NScan): Check invalid usage of l10n strings
5. [LogToZMLog](#LogToZMLog): Should not use android.util.Log
6. [LogWithinDebug](#LogWithinDebug): Should not use ZMLog outside if(BuildConfig.LOG_ENABLE)
7. [RawStringTest](#RawStringTest): Raw string contains "test" is not allowed
8. [StartActivity](#StartActivity): Should not use startActivity directly
9. [OldTargetApi](#OldTargetApi): Target SDK attribute is not targeting latest version
10. [EllipsizeMaxLines](#EllipsizeMaxLines): Combining Ellipsize and Maxlines
11. [StringFormatInvalid](#StringFormatInvalid): Invalid format string
12. [StringFormatMatches](#StringFormatMatches): `String.format` string doesn't match the XML format string
13. [PluralsCandidate](#PluralsCandidate): Potential Plurals
14. [StringFormatCount](#StringFormatCount): Formatting argument types incomplete or inconsistent
15. [SyntheticAccessor](#SyntheticAccessor): Synthetic Accessor
15. [NewApi](#NewApi): Calling new methods on older versions

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

<gradient> 在api24以下可能无法解析导致crash，如果drawable资源文件中有这样的用法需要同时提供用在api21设备上的同名资源文件（文件中没有使用到<gradient>），或者不使用<gradient> 

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:aapt="http://schemas.android.com/aapt"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
  <path
      android:pathData="M6.4806,8.3938C4.4996,8.3938 "
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

### 1.9 OldTargetApi

targetSdkVersion为更新到最新版

```
defaultConfig {
    applicationId "us.zoom.sample"
    minSdkVersion 21
    targetSdkVersion 30
```

### 1.10 EllipsizeMaxLines

同时使用`android:maxLines="1"`和`android:ellipsize="middle"`可能会导致低版本crash，请使用`singleLine=true`，`singleLine`被废弃但是依然可以使用

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:maxLines="1"                                                                        
    android:ellipsize="middle"
    android:layout_height="wrap_content" />
```

### 1.11 StringFormatInvalid && StringFormatMatches && StringFormatCount

string参数不规范，需要保证参数数量、类型、顺序与调用处相同

```xml
<string name="invalid_format_string">My name is %s , and I\'m %d age,%</string>
```

```java
Toast.makeText(this, String.format(getString(R.string.invalid_format_string), 20, "Flueky"), Toast.LENGTH_LONG).show();
```

### 1.12 PluralsCandidate

string中存在单复数情况，`new messages`，可能需要使用plurals

```xml
<string name="formatting_strings">Hello, %1$s! You have %2$d new messages. %3$d unread</string>
```

### 1.13 SyntheticAccessor

访问内部类的静态成员，该成员不能用`private`修饰，`MyCallback`需要用`protected`或者`public`修饰

```java
public class HookAMS {
    // ...
    public static void hookActivityThread() {
        try {
            // XXX，访问内部类的静态成员，该成员不能用`private`修饰
            mCallbackField.set(mH, new MyCallback());
        } catch (ClassNotFoundException | NoSuchFieldException | NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }

    private static class MyCallback implements Handler.Callback {
        @Override
        public boolean handleMessage(@NonNull Message msg) {
            Object clientTransactionObj = msg.obj;
            try {
                // XXX
            } catch (ClassNotFoundException | NoSuchFieldException | IllegalAccessException e) {
                e.printStackTrace();
            }
            return false;
        }
    }
}
```

### 1.14 NewApi

`String.join`api26以上才能使用，api21设备会crash，用其他方法替换

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    String.join("", "");
}
```


## 2. 自定义lint

首先需要clone https://git.zoom.us/main/android_cn仓库，代码在[lintchecks][https://git.zoom.us/main/android_cn/-/tree/master/lintchecks]项目中，lintchecks工程中有3个module: `checks`, `library`, `sample`，其中`library`不用管，`sample`用于编写用来测试lint功能的代码，`checks`用于编写自定义lint，以`DebugAndLogDetector`为例

```java
public class DebugAndLogDetector extends Detector implements SourceCodeScanner {
```

这个DebugAndLogDetector用于扫描`DebugToLogEnable`, `LogToZMLog`, `StartActivity`, `LogWithinDebug`4个规则，这4个规则都是扫描java代码或者kotlin代码，因此直接继承Detector并实现SourceCodeScanner接口即可，如果需要扫描的是xml资源文件，则继承ResourceXmlDetector，具体可参考SVGDetector。

然后需要定义一个Issue，用于表示此lint规则的相关信息，比如危险等级、概要、解释、id、优先级、分类等，参数说明参考[自定义 Lint 检查实践指南](https://zhuanlan.zhihu.com/p/307382854)

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

然后将DebugAndLogDetector注册到ZoomIssueRegistry中

```java
public class ZoomIssueRegistry extends IssueRegistry {
    @NotNull
    @Override
    public List<Issue> getIssues() {
        List<Issue> issueList = new ArrayList<>();
        
        issueList.add(DebugAndLogDetector.ISSUE_LOG);
        issueList.add(DebugAndLogDetector.ISSUE_DEBUG);
        issueList.add(DebugAndLogDetector.ISSUE_ACTIVITY);

        return issueList;
    }

    @Override
    public int getApi() {
        return ApiKt.CURRENT_API;
    }
}
```

最后就是编写扫描代码的规则，以`DebugToLogEnable`规则为例，我们需要扫描到代码里的`BuildConfig.DEBUG`字段，如果扫描则将对应的文件名、行号信息输出；如何定位，这里使用了AST与PSI规则，简而言之就是对一个文本文件进行结构化分析，取出对应的字段保存在某个对象中，具体定义可以参考[Implementing a Parser and PSI](https://kana112233.github.io/intellij-sdk-docs-cn/reference_guide/custom_language_support/implementing_parser_and_psi.html)

这里我们需要重写`getApplicableReferenceNames`用于查找引用，返回值为需要查询的字段名称

```java
@Nullable
@Override
public List<String> getApplicableReferenceNames() {
    return Arrays.asList("DEBUG", "BuildConfig");
}
```

通过`getApplicableReferenceNames`定义好的查询字段，lint运行起来后会调用`visitReference`处理查询到的结果，所以重写`visitReference`方法

```java
@Override
public void visitReference(@NotNull JavaContext context, @NotNull UReferenceExpression reference, @NotNull PsiElement referenced) {
    // reference是用于保存查询字段的对象
    if ("DEBUG".equals(reference.getResolvedName())){
        // 通过这个对象查询父节点，并拿到父节点的name，即DEBUG的类名
        PsiElement parent = referenced.getParent();
        String name = ((PsiClassImpl) parent).getName();
        // 拿到类名直接判断即可
        if ("BuildConfig".equals(name)){
            context.report(ISSUE_DEBUG, reference, context.getNameLocation(reference)
                    , "change \"BuildConfig.DEBUG\" to \"BuildConfig.LOG_ENABLE\"");
        }
    }
}
```

在某些场景中，需要扫描的结构比较复杂，PSI结构不明确，可以在Android Studio中安装**PsiViewer**插件，这个可以直接分析出文件的PSI结构，便于前期代码编写，以 `LogWithinDebug`为例，首先需要定义查找的字段，不同于`DebugToLogEnable`， `LogWithinDebug`需要查找到某些方法（`ZMLog.i, ZMLog.d`）的使用位置

```java
@Nullable
@Override
public List<String> getApplicableMethodNames() {
    return Arrays.asList(
            "d",
            "e",
            "i",
            "v",
            "w"
    );
}
```

```java
@Override
public void visitMethodCall(@NotNull JavaContext context, @NotNull UCallExpression node, @NotNull PsiMethod method) {
    JavaEvaluator evaluator = context.getEvaluator();
    // isMemberInClass可以判断方法所在的类
    if (evaluator.isMemberInClass(method, ZMLOG_CLS)) {
        if (!isWrappedByLogEnable(node, context)) {
            context.report(ISSUE_LOG_WITHIN_LOG_ENABLE, node, context.getNameLocation(node)
                    , "Wrap ZMLog with if(BuildConfig.LOG_ENABLE)");
        }
    }
}
```

```java
// isWrappedByLogEnable判断ZMLog.i方法是否在BuildConfig.LOG_ENABLE条件中
private boolean isWrappedByLogEnable(UCallExpression node, JavaContext context) {
    boolean result = false;
    // kotlin PSI结构与 java有一些区别，但是依然可以使用PsiViewer来解析
    if (node instanceof KotlinUFunctionCallExpression) {
        for (UElement parent = node.getUastParent(); parent != null; parent = parent.getUastParent()) {
            try {
                if (!(parent instanceof KotlinUIfExpression)){
                    continue;
                }
                KtExpression condition = ((KotlinUIfExpression) parent).getSourcePsi().getCondition();
                if (condition.getText().contains("BuildConfig.LOG_ENABLE") || condition.getText().contains("us.zoom.androidlib.BuildConfig.LOG_ENABLE")){
                    result = true;
                    break;
                }
            } catch (Exception e) {

            }
        }
    } else {
        // 根据PsiViewer的结构分析，可以发现只要一直找父节点就可以拿到PsiIfStatement，这个恰好表示的是If语句，那么只需要拿到条件表达式并与BuildConfig.LOG_ENABLE比对即可，同理可以对kotlin文件进行判断
        for (PsiElement parent = node.getSourcePsi().getParent(); parent != null && !(parent instanceof MethodElement); parent = parent.getParent()) {
            if (parent instanceof PsiIfStatement) {
                PsiExpression condition = ((PsiIfStatement) parent).getCondition();
                if (condition.getText().contains("BuildConfig.LOG_ENABLE") || condition.getText().contains("us.zoom.androidlib.BuildConfig.LOG_ENABLE")){
                    result = true;
                    break;
                }
            }
        }
    }
    return result;
}
```

![Screen Shot 2021-12-21 at 21.58.19](https://gitee.com/tao2333/hugo-pic/raw/master/pictures/202112212201772.png)

如果需要debug这些Detector的代码，需要新增**Remote**的Configurations，全部默认值即可，然后在项目根目录执行以下命令，参数`:sample:lintDebug`表示需要执行的是sample module的lintDebug task

```
gradlew --no-daemon -Dorg.gradle.debug=true :sample:lintDebug
```

## 3. 部署

完成lint代码后需要修改`gradle.properties`，将`VERSION = 1.0.3`版本号增加，否则将无法部署，然后提交到android_cn仓库中，之后通过[jenkins build](https://jenkins.client.corp.zoom.us/view/Android/job/Client/job/Client/job/lintchecks/)，这个阶段会将这个library部署到**zoom-mvn-virtual**仓库中，部署成功后在项目中就可以使用了，修改androidlib的build.gradle，更新版本号为`VERSION `字段的值

```groovy
api "us.zoom.lintchecks:library:1.0.3"
```

## 4. 使用

项目根目录执行以下命令即可扫描videobox中的代码，或者在Android Studio中右侧边栏使用Gradle -> zoom -> videobox -> Tasks -> verification -> :videobox:lintRelease

```
gradlew :videobox:lintRelease
```

如果需要忽略lint的扫描结果，有以下几种方式：

1. With a `@SuppressLint` annotation in the Java code
2. With a `tools:ignore` attribute in the XML file
3. With a //noinspection comment in the source code
4. With ignore flags specified in the `build.gradle` file, as explained below
5. With a `lint.xml` configuration file in the project
6. With a `lint.xml` configuration file passed to lint via the --config flag
7. With the --ignore flag passed to lint.

## 参考

1. [自定义 Lint 检查实践指南](https://zhuanlan.zhihu.com/p/307382854)
2. [Android自定义Lint实践](https://tech.meituan.com/2016/03/21/android-custom-lint.html)
3. [android自定义Lint实现代码检测](https://juejin.cn/post/6844903774104879111)
4. [googlesamples/android-custom-lint-rules](https://github.com/googlesamples/android-custom-lint-rules)
4. [Implementing a Parser and PSI](https://kana112233.github.io/intellij-sdk-docs-cn/reference_guide/custom_language_support/implementing_parser_and_psi.html)
