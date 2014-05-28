<!---
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->

# org.apache.cordova.file

这个插件提供了[HTML5 文件系统 API][1]。 用法，请参阅 HTML5 岩石[文件系统条][2]关于这个问题。 其他存储选项的概述，请参阅科尔多瓦的[存储指南][3].

 [1]: http://dev.w3.org/2009/dap/file-system/pub/FileSystem/
 [2]: http://www.html5rocks.com/en/tutorials/file/filesystem/
 [3]: http://cordova.apache.org/docs/en/edge/cordova_storage_storage.md.html

## 安装

    cordova plugin add org.apache.cordova.file
    

## 支持的平台

*   亚马逊火 OS
*   Android 系统
*   黑莓 10
*   iOS
*   Windows Phone 7 和 8 *
*   Windows 8 *
*   火狐浏览器操作系统

**这些平台不支持 `FileReader.readAsArrayBuffer` ，也不 `FileWriter.write(blob)` .*

## 存储文件的位置

自 v1.2.0，提供重要的文件系统目录的 Url。 每个 URL 是在窗体*file:///path/to/spot/*，和可以转换为 `DirectoryEntry` 使用`window.resolveLocalFileSystemURL()`.

`cordova.file.applicationDirectory`-只读目录应用程序的安装位置。(*iOS* *Android*)

`cordova.file.applicationStorageDirectory`-应用程序的私有可写存储的根。(*iOS* *Android*)

`cordova.file.dataDirectory`-在何处放置应用程序特定的数据文件。(*iOS* *Android*)

`cordova.file.cacheDirectory`-缓存应该生存重新启动应用程序的文件。应用程序不应依赖 OS，以删除文件在这里。(*iOS* *Android*)

`cordova.file.externalApplicationStorageDirectory`-应用程序外部存储上的空间。(*iOS* *Android*)

`cordova.file.externalDataDirectory`-要把外部存储的特定于应用程序数据文件的位置。(*Android*)

`cordova.file.externalCacheDirectory`外部存储的应用程序缓存。(*Android*)

`cordova.file.externalRootDirectory`-外部存储 （SD 卡） 根。(*Android*)

`cordova.file.tempDirectory`-将 OS 可以清除时的 temp 目录。(*iOS*)

`cordova.file.syncedDataDirectory`-持有应同步 （例如到 iCloud) 的应用程序特定的文件。(*iOS*)

`cordova.file.documentsDirectory`-文件专用的应用程序，但这是对其他 applciations （例如 Office 文件） 有意义。(*iOS*)

## Android 的怪癖

### Android 的永久存储位置

有多个存储持久性 Android 设备上的文件的有效位置。 请参阅[此页][4]为广泛地讨论的各种可能性。

 [4]: http://developer.android.com/guide/topics/data/data-storage.html

以前版本的插件会选择在启动时，基于是否该设备声称 SD 卡 （或等效存储分区） 安装临时和永久文件的位置。 如果被挂载 SD 卡，或者如果一个大的内部存储分区是可用 （如 Nexus 的设备上，） 然后持久性文件将存储在该空间的根。 这就意味着所有的科尔多瓦应用可以看到所有可用的文件在卡上。

如果 SD 卡不是可用的然后以前的版本会将数据存储在下/数据/数据 /<packageid>其中隔离应用程序从彼此，但仍然可能会导致用户之间共享数据。

现在可以选择是否要将文件存储在内部文件存储位置，或使用以前的逻辑，在您的应用程序的 config.xml 文件首选项。 若要执行此操作，请将这两行之一添加到 config.xml：

    <preference name="AndroidPersistentFileLocation" value="Internal" />
    
    <preference name="AndroidPersistentFileLocation" value="Compatibility" />
    

如果这条线，没有文件插件将作为默认值使用"兼容性"。如果首选项标记是存在，并不是这些值之一，应用程序将无法启动。

如果您的应用程序先前已经运送到用户，使用较旧的 (预 1.0) 的这个插件版本和已存储的文件中的持久性的文件系统，然后您应该设置的首选项的"兼容性"。 切换到"内部"的位置就意味着现有用户升级他们的应用程序可能无法访问他们以前存储的文件，他们的设备。

如果您的应用程序是新的或以前从未有持久性文件系统中存储的文件，然后通常会建议的"内部"的设置。

## iOS 的怪癖

*   `FileReader.readAsText(blob, encoding)` 
    *   `encoding`参数不受支持，和 utf-8 编码总是有效。

### iOS 的持久性存储位置

有两个有效位置来存储持久性的 iOS 设备上的文件： 文件目录和库目录。 以前版本的插件只过持久性文件存储在文件目录中。 这有副作用 — — 使所有的应用程序的文件可见在 iTunes 中，往往是意料之外的特别是为处理大量小文件的应用程序，而不是生产用于出口，是预期的目的的目录的完整的文件。

现在可以选择是否要将文件存储在文件或库目录，与您的应用程序的 config.xml 文件中的偏好。 若要执行此操作，请将这两行之一添加到 config.xml：

    <preference name="iosPersistentFileLocation" value="Library" />
    
    <preference name="iosPersistentFileLocation" value="Compatibility" />
    

如果这条线，没有文件插件将作为默认值使用"兼容性"。如果首选项标记是存在，并不是这些值之一，应用程序将无法启动。

如果您的应用程序先前已经运送到用户，使用较旧的 (预 1.0) 的这个插件版本和已存储的文件中的持久性的文件系统，然后您应该设置的首选项的"兼容性"。 切换到"库"的位置意味着升级他们的应用程序的现有用户将无法访问他们以前存储的文件。

如果您的应用程序是新的或以前从未有持久性文件系统中存储的文件，然后通常会建议"的库"设置。

### 火狐浏览器操作系统的怪癖

文件系统 API 本机操作系统不支持火狐浏览器，在 indexedDB shim 作为实现的。

*   不会失败时删除非空的目录
*   不支持用元数据的目录
*   不支持 `requestAllFileSystems` 和 `resolveLocalFileSystemURI` 的方法
*   方法 `copyTo` 和 `moveTo` 不支持目录

## 升级说明

在此插件 v1.0.0 `FileEntry` 和 `DirectoryEntry` 结构已经更改，以更符合已发布的规范。

以前 (pre-1.0.0） 版本的插件存储的设备-绝对-文件-位置在 `fullPath` 属性的 `Entry` 对象。这些路径通常看起来就像

    /var/mobile/Applications/<application UUID>/Documents/path/to/file  (iOS)
    /storage/emulated/0/path/to/file                                    (Android)
    

这些路径还返回的 `toURL()` 方法的 `Entry` 对象。

与 v1.0.0， `fullPath` 属性是对该文件*的 HTML 文件系统根目录的相对*路径。 因此，上述路径会现在两者都由表示 `FileEntry` 对象与 `fullPath` 的

    /path/to/file
    

如果您的应用程序与设备-绝对路径，和你以前检索到这些路径通过 `fullPath` 属性的 `Entry` 对象，然后你应该更新您的代码以使用 `entry.toURL()` 相反。

为向后兼容性， `resolveLocalFileSystemURL()` 方法将接受绝对设备的路径，并将返回 `Entry` 对应于它，只要该文件存在的临时或永久性的文件系统内的对象。

这特别是一直带有文件传输插件，以前使用过的设备绝对路径的问题 (和仍然可以接受他们)。 它已更新为与文件系统的 Url，所以取代正常工作 `entry.fullPath` 与 `entry.toURL()` 应解决获取该插件来处理文件的设备上的任何问题。

V1.1.0 的返回值中的 `toURL()` 已更改 （请参见 \[CB-6394\] (https://issues.apache.org/jira/browse/CB-6394)） 返回一个绝对 'file:// URL。 在可能的情况。 为确保 ' cdvfile:'-您可以使用的 URL `toInternalURL()` 现在。 现在，此方法将返回该窗体的文件系统 Url

    cdvfile://localhost/persistent/path/to/file
    

它可以用于唯一地标识该文件。

## 错误代码和含义的列表

错误引发时，将使用以下代码之一。

*   1 = NOT\_FOUND\_ERR
*   2 = SECURITY_ERR
*   3 = ABORT_ERR
*   4 = NOT\_READABLE\_ERR
*   5 = ENCODING_ERR
*   6 = NO\_MODIFICATION\_ALLOWED_ERR
*   7 = INVALID\_STATE\_ERR
*   8 = SYNTAX_ERR
*   9 = INVALID\_MODIFICATION\_ERR
*   10 = QUOTA\_EXCEEDED\_ERR
*   11 = TYPE\_MISMATCH\_ERR
*   12 = PATH\_EXISTS\_ERR

## 配置的插件 （可选）

可用的文件系统的一组可配置的每个平台。IOS 和 android 系统识别 <preference> 在中添加标签 `config.xml` 的名字要安装的文件系统。默认情况下，启用所有文件系统根。

    <preference name="iosExtraFilesystems" value="library,library-nosync,documents,documents-nosync,cache,bundle,root" />
    <preference name="AndroidExtraFilesystems" value="files,files-external,documents,sdcard,cache,cache-external,root" />
    

### Android 系统

*   文件: 应用程序的内部文件存储目录
*   文件外部： 应用程序的外部文件存储目录
*   sdcard： 全球外部文件存储目录 （如果安装了一个，这是 SD 卡的根目录）。 您必须具有 `android.permission.WRITE_EXTERNAL_STORAGE` 使用此权限。
*   高速缓存： 应用程序的内部缓存目录
*   外部高速缓存： 应用程序的外部高速缓存目录
*   根： 整个设备的文件系统

Android 还支持名为"文件"，它表示"文件的"文件系统中的子目录"/ 文件 /"特别的文件系统。

### iOS

*   图书馆： 应用程序的库目录
*   文档： 应用程序的文件目录
*   高速缓存： 应用程序的缓存目录
*   束： 束应用程序的 ；应用程序本身 （只读） 的磁盘上的位置
*   根： 整个设备的文件系统

默认情况下，图书馆和文件目录可以同步到 iCloud。 你也可以要求两个附加文件系统、"图书馆-非同步"和"文档-nosync"，代表特别的非同步的目录内的库或文件的文件系统。