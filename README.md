
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
在app目录下的build.gradle引用 dialogxVersion最新版本为 0.0.50.beta38
```gradle
    val dialogXVersion = "0.0.50.beta38"
    //引入DialogX主体
    implementation("com.github.suzhelan.DialogX:DialogX:$dialogXVersion")
    //非必须 DialogX官方提供的主题样式
    implementation("com.github.suzhelan.DialogX:DialogXKongzueStyle:$dialogXVersion")
    implementation ("com.github.suzhelan.DialogX:DialogXMIUIStyle:$dialogXVersion")
    implementation("com.github.suzhelan.DialogX:DialogXIOSStyle:$dialogXVersion")
    implementation("com.github.suzhelan.DialogX:DialogXMaterialYou:$dialogXVersion")
```

初始化流程正常初始化即可,不要忘了将模块APK资源注入到宿主Activity，不然会查找不到XML的  
一般建议在 Application 的实现类中的 onCreate 方法中进行配置：
```java
        //初始化
        DialogX.init(this);
        DialogX.globalTheme = DialogX.THEME.AUTO;
        DialogX.globalStyle = new MaterialYouStyle();//如果有
```
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
### 疑难解决   
- 1.如果遇到在Dialog里使用ListView,RecyclerView等动态长度的View,始终只展示一行的高度 可通过以下代码解决 **注意 这是Dialogx自身的特性,与本fork无关**
```kotlin
        MessageDialog.build()
            .setTitle("请选择赞助项目")
            .onShow {
                val listView = SimpleTextListAdapter.createView(
                    activity,
                    singleSelectMenuText,
                    object : SimpleTextListAdapter.OnItemClickListener {
                        override fun onItemClick(
                            adapter: BaseAdapter,
                            position: Int,
                            view: View
                        ) {
                            //跳转到浏览器
                            val intent = Intent(Intent.ACTION_VIEW)
                            intent.data = payItemList[position].payUrl.toUri()
                            activity.startActivity(intent)
                        }
                    }
                )
                //核心代码
                it.dialogImpl.boxList.addView(listView)
                it.dialogImpl.boxList.isVisible = true
            }
            .setMessage(message)
            .setOkButton("支付完成") { dialog, v ->
                showQueryOrderResultDialog()
                false
            }.show()
```
