# Android studio release jar by gradle

> 原文網址：https://boochlin.com/?p=84
> 發布日期：2015-10-05
> 文章編號：84

---

### Android studio release jar by gradle （test on 1.3）

主要來說android build的時候就會在 build/build/intermediates/bundles/release 放置classes.jar

但是很多人可能不清楚或忘記，所以這個script只是把 classes.jar copy 到 build/libs 下

並 rename 成 團隊發布的vname.jar，

目前底下的code已經符合要求

```
def versionPropsFile = file('../version.properties')
task releaseJar(type: Copy) {
    if (versionPropsFile.canRead()) {
        def Properties versionProps = new Properties();
        versionProps.load(new FileInputStream(versionPropsFile));
        def vName = versionProps['VERSION_NAME'];
        def vCode = versionProps['VERSION_CODE'];
        def int vnCode = vCode.toInteger()
        println sprintf("vName: %s vCode %d", vName, vnCode);

        from( 'build/intermediates/bundles/release')
        into( 'build/libs')
        include('classes.jar')
        rename('classes.jar', 'home-'+ vName + '.jar')

    } else {
        throw new GradleException("Could not read version.properties!")
    }
}
```

如果要加入 proguard 可以參考

[http://chaosleong.github.io/blog/2015/08/02/android-studio-shi-yong-gradle-da-bao-jar/](http://chaosleong.github.io/blog/2015/08/02/android-studio-shi-yong-gradle-da-bao-jar/)
