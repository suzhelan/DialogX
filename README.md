
## [官方文档](https://github.com/kongzue/DialogX/tree/master)

## 本分支修改 ，适配在XPosed环境中宿主使用的使用问题

使用指南
在项目根目录 settings.gradle 添加 jitpack
```gradle
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        mavenLocal()
        maven { url = uri("https://jitpack.io") }
    }
}
```
在app目录下的build.gradle引用
```gradle
    //引入DialogX主体
    implementation("com.github.suzhelan.DialogX:DialogX:0.0.50.beta23")
    //非必须 DialogX官方提供的主题样式
    implementation("com.github.suzhelan.DialogX:DialogXKongzueStyle:0.0.50.beta23")
    implementation ("com.github.suzhelan.DialogX:DialogXMIUIStyle:0.0.50.beta23")
    implementation("com.github.suzhelan.DialogX:DialogXIOSStyle:0.0.50.beta23")
```

初始化流程正常初始化即可,不要忘了将模块APK资源注入到宿主Activity，不然会查找不到XML的  
附上注入模块APK注入资源到宿主的代码
```java
    public static void injectResourcesToContext(Context context) {
        try {
            Resources resources = context.getResources();
            AssetManager assetManager = resources.getAssets();
            @SuppressLint("DiscouragedPrivateApi")
            Method method = AssetManager.class.getDeclaredMethod("addAssetPath", String.class);
            method.setAccessible(true);
            method.invoke(assetManager, moduleApkPath);
        } catch (Exception ex) {
        }
    }
```
