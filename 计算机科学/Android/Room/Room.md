### [Google的手册](https://developer.android.google.cn/training/data-storage/room?hl=zh-cn)

一种高度集成的数据库。
# 介绍
## 框架
Room 包含三个主要组件：

- [数据库类](https://developer.android.google.cn/reference/kotlin/androidx/room/Database?hl=zh-cn)，用于保存数据库并作为应用持久性数据底层连接的主要访问点。
- [数据实体](https://developer.android.google.cn/training/data-storage/room/defining-data?hl=zh-cn)，用于表示应用的数据库中的表。
- [数据访问对象 (DAO)](https://developer.android.google.cn/training/data-storage/room/accessing-data?hl=zh-cn)，为您的应用提供在数据库中查询、更新、插入和删除数据的方法。

数据库类为应用提供与该数据库关联的 DAO 的实例。反过来，应用可以使用 DAO 从数据库中检索数据，作为关联的数据实体对象的实例。此外，应用还可以使用定义的数据实体更新相应表中的行，或者创建新行供插入。图 1 说明了 Room 的不同组件之间的关系。

# 使用
## 前置依赖
在顶级 `build.gradle.kts`
```kotlin
plugins {
    id("com.google.devtools.ksp") version "2.3.4" apply false
}
```


在模块级 `build.gradle.kts`
```kotlin
plugins {
    id("com.google.devtools.ksp")
}
```

```kotlin
dependencies {
    val room_version = "2.8.4"

    implementation("androidx.room:room-runtime:$room_version")

    // If this project uses any Kotlin source, use Kotlin Symbol Processing (KSP)
    // See [Add the KSP plugin to your project](https://developer.android.google.cn/build/migrate-to-ksp?hl=zh-cn#add-ksp)
    ksp("androidx.room:room-compiler:$room_version")

    // If this project only uses Java source, use the Java annotationProcessor
    // No additional plugins are necessary
    annotationProcessor("androidx.room:room-compiler:$room_version")

    // optional - Kotlin Extensions and Coroutines support for Room
    implementation("androidx.room:room-ktx:$room_version")

    // optional - RxJava2 support for Room
    implementation("androidx.room:room-rxjava2:$room_version")

    // optional - RxJava3 support for Room
    implementation("androidx.room:room-rxjava3:$room_version")

    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation("androidx.room:room-guava:$room_version")

    // optional - Test helpers
    testImplementation("androidx.room:room-testing:$room_version")

    // optional - Paging 3 Integration
    implementation("androidx.room:room-paging:$room_version")
}
```
## 