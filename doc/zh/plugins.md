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

# 插件开发人员须知

这些笔记主要针对 Android 和 iOS 开发者想要使用的文件系统使用的文件插件编写插件哪个接口。

## 工作与科尔多瓦文件系统 Url

1.0.0 版以来，这个插件用了 Url 与 `cdvfile` 在桥梁，为所有的通信计划，而不是暴露 JavaScript 原始设备的文件系统路径。

在 JavaScript 方面，这意味着 FileEntry 和 DirectoryEntry 的对象具有一个完整路径属性是相对于 HTML 文件系统的根目录。 如果你的插件的 JavaScript API 接受 FileEntry 或 DirectoryEntry 的对象，则应调用 `.toURL()` 对该对象之前将它在桥上传递给本机代码。

### 转换 cdvfile: / / fileystem 的路径的 Url

需要写入到文件系统的插件可能会想要将收到的文件系统的 URL 转换为实际的文件系统位置。有多种方法做这个，根据本机平台。

它是重要的是要记住，不是所有 `cdvfile://` 的 Url 均可映射到设备上的实际文件。 某些 Url 可以引用在设备上不是由文件，或甚至可以引用远程资源的资产。 由于这些可能性，插件应始终测试是否回试图将 Url 转换为路径时，他们得到有意义的结果。

#### Android 系统

在 android 系统，最简单的方法来转换 `cdvfile://` 文件系统路径的 URL 是使用 `org.apache.cordova.CordovaResourceApi` 。 `CordovaResourceApi`有几种方法，可处理 `cdvfile://` 的 Url：

    // webView is a member of the Plugin class
    CordovaResourceApi resourceApi = webView.getResourceApi();
    
    // Obtain a file:/// URL representing this file on the device,
    // or the same URL unchanged if it cannot be mapped to a file
    Uri fileURL = resourceApi.remapUri(Uri.parse(cdvfileURL));
    

它也是可以直接使用文件插件：

    import org.apache.cordova.file.FileUtils;
    import org.apache.cordova.file.FileSystem;
    import java.net.MalformedURLException;
    
    // Get the File plugin from the plugin manager
    FileUtils filePlugin = (FileUtils)webView.pluginManager.getPlugin("File");
    
    // Given a URL, get a path for it
    try {
        String path = filePlugin.filesystemPathForURL(cdvfileURL);
    } catch (MalformedURLException e) {
        // The filesystem url wasn't recognized
    }
    

要转换的路径从 `cdvfile://` 的 URL：

    import org.apache.cordova.file.LocalFilesystemURL;
    
    // Get a LocalFilesystemURL object for a device path,
    // or null if it cannot be represented as a cdvfile URL.
    LocalFilesystemURL url = filePlugin.filesystemURLforLocalPath(path);
    // Get the string representation of the URL object
    String cdvfileURL = url.toString();
    

如果你的插件创建一个文件，并且您想要为它返回一个 FileEntry 对象，使用该文件插件：

    // Return a JSON structure suitable for returning to JavaScript,
    // or null if this file is not representable as a cdvfile URL.
    JSONObject entry = filePlugin.getEntryForFile(file);
    

#### iOS

科尔多瓦在 iOS 上的不使用相同 `CordovaResourceApi` 作为 android 系统的概念。在 iOS，应使用文件插件的 Url 和文件系统路径之间进行转换。

    // Get a CDVFilesystem URL object from a URL string
    CDVFilesystemURL* url = [CDVFilesystemURL fileSystemURLWithString:cdvfileURL];
    // Get a path for the URL object, or nil if it cannot be mapped to a file
    NSString* path = [filePlugin filesystemPathForURL:url];
    
    
    // Get a CDVFilesystem URL object for a device path, or
    // nil if it cannot be represented as a cdvfile URL.
    CDVFilesystemURL* url = [filePlugin fileSystemURLforLocalPath:path];
    // Get the string representation of the URL object
    NSString* cdvfileURL = [url absoluteString];
    

如果你的插件创建一个文件，并且您想要为它返回一个 FileEntry 对象，使用该文件插件：

    // Get a CDVFilesystem URL object for a device path, or
    // nil if it cannot be represented as a cdvfile URL.
    CDVFilesystemURL* url = [filePlugin fileSystemURLforLocalPath:path];
    // Get a structure to return to JavaScript
    NSDictionary* entry = [filePlugin makeEntryForLocalURL:url]
    

#### JavaScript

在 JavaScript 中，得到 `cdvfile://` URL 从 FileEntry 或 DirectoryEntry 的对象，只需调用 `.toURL()` 对它：

    var cdvfileURL = entry.toURL();
    

在插件响应处理程序，将从返回的 FileEntry 结构转换为实际的输入对象，您的处理程序代码应该导入的文件插件并创建一个新的对象：

    // create appropriate Entry object
    var entry;
    if (entryStruct.isDirectory) {
        entry = new DirectoryEntry(entryStruct.name, entryStruct.fullPath, new FileSystem(entryStruct.filesystemName));
    } else {
        entry = new FileEntry(entryStruct.name, entryStruct.fullPath, new FileSystem(entryStruct.filesystemName));
    }