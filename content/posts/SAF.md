---
title: "Android-Q适配-存储方式"
date: 2019-10-26T11:02:06+08:00
tags: ["Android Q"]
categories: ["Android"]
series: [""]
summary: "Android Q之后对系统存储方式进行了调整，简而言之就是禁止开发人员随意通过路径访问操作外部存储文件，内部存储没有影响。这样做的目的很明显，即往后原生Android的文件管理器将不会出现各种App生成的乱七八糟的文件，不同类型的文件都在其各自相应的位置。"
draft: false
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

Android Q之后对系统存储方式进行了调整，简而言之就是禁止开发人员随意通过路径访问操作外部存储文件，内部存储没有影响。这样做的目的很明显，即往后原生Android的文件管理器将不会出现各种App生成的乱七八糟的文件，不同类型的文件都在其各自相应的位置。

示意图如下，主要行为变更在媒体文件（音频、视频、图片）以及下载文件中

![file](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/file.png)

## 1. SAF框架

SAF（Storage Access Framework，存储访问框架），是Android 4.4之后提供的文件选择器，通过Intent方式启动，UI界面由系统提供，一般来说其他厂商魔改的系统都没有对这方面进行重写，所以示意图基本相同，如下

![saf](https://cdn.jsdelivr.net/gh/Zhoutao822/hugo-pic/pictures/saf.png)

### 1.1 外部存储

如果应用有发送文件、选择文件等功能，特别是需要读取系统文件目录的方法都需要修改，在target Q的情况下，试图通过路径访问外部公共文件的方式都会失效，如下所示，因此需要使用SAF框架选取文件，得到的结果是文件的Uri，然后再使用Uri读取或者处理该文件，外部存储路径如果打印出来类似`/storage/emulated/0/Android/data/us.zoom.androidqdemo/files/aabb.rar`、`/storage/emulated/0/Pictures/Screenshots/Screenshot_20191014-141713.png`，前者是当前应用的外部路径（可以直接访问），后者是公共图片文件下外部路径；内部路径类似`/data/user/0/us.zoom.androidqdemo/data/aabb.png`，内部路径无法直接在系统中查看。

```java
// 在手机外部存储根目录下创建text.txt文件，在Android P上可以成功创建此文件并写入数据，file.exists()返回True；
// 在Android Q上无法创建此文件，返回False
String filename = "text.txt";
File file = new File(Environment.getExternalStorageDirectory(), filename);
// File file = new File(getFilesDir(), filename); //如果是内部存储，则两者都是正常访问
// File file = new File(getExternalFilesDir(""), filename); // 外部存储的当前应用路径可以正常访问
try {
    FileOutputStream outputStream = new FileOutputStream(file);
    outputStream.write("123456".getBytes());
    outputStream.close();
} catch (Exception e) {
    e.printStackTrace();
}
Toast.makeText(this, file.exists() ? "True" : "False", Toast.LENGTH_SHORT).show();
```

```java
// 同理对读文件，Android P上可以读取text.txt内容，并且file.exists()返回True；Android Q上FileInputStream报错：
// 无法访问，拒绝权限，但是file.exists()返回true
String filename = "text.txt";
File file = new File(Environment.getExternalStorageDirectory(), filename);
// File file = new File(getFilesDir(), filename); //如果是内部存储，则两者都是正常访问
// File file = new File(getExternalFilesDir(""), filename); // 外部存储的当前应用路径可以正常访问
try {
    FileInputStream inputStream = new FileInputStream(file);
    InputStreamReader reader = new InputStreamReader(inputStream);
    BufferedReader br = new BufferedReader(reader);
    String line = "";
    line = br.readLine();
    while (line != null) {
        Log.i("TestRead", line);
        line = br.readLine(); // 一次读入一行数据
    }
} catch (Exception e) {
    e.printStackTrace();
}
Toast.makeText(this, file.exists() ? "True" : "False", Toast.LENGTH_SHORT).show();
```

### 1.2 SAF使用

SAF原理可以在Google官网[使用存储访问框架打开文件](https://developer.android.com/guide/topics/providers/document-provider?hl=zh-cn)中查看，发送文件不能通过公共路径，那么就需要使用SAF。SAF读取到的文件有四类：图片、音频、视频、下载、内部存储空间（外部存储）以及各种网盘，可以通过setType设置显示的文件类别，其中图片（一般包含根目录以及照片DCIM文件夹、公有图片Pictures文件夹、Download下图片文件）、音频（一般包含根目录以及公有音频Music文件夹、Download下音频文件）、视频（一般包含根目录以及视频DCIM文件夹、公有视频Movies文件夹、Download下视频文件）中都是通过MediaStore保存的文件，下载中都是通过DownloadManager下载的文件，否则不显示在这几个目录中；内部存储空间即外部存储，从这里可以访问各个应用的外部存储；各种网盘也可以访问，从网盘获取文件会先调用网盘的下载功能，然后再获取下载好的文件，下载过程不可见。

```java
// 4.3及以下用 ACTION_PICK 或 ACTION_GET_CONTENT，Android 4.4以上可以多一个选择ACTION_OPEN_DOCUMENT，ACTION_PICK弹出单项选择窗口
// ACTION_GET_CONTENT与ACTION_OPEN_DOCUMENT类似，且使用ACTION_GET_CONTENT时，应用会导入数据（如图片文件）的副本，即如果
// 只是需要读取数据而不修改原始数据，那就用ACTION_GET_CONTENT，如果需要修改，使用ACTION_OPEN_DOCUMENT
Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
// 过滤器只显示可以打开的结果
intent.addCategory(Intent.CATEGORY_OPENABLE);
// 使用图像MIME数据类型过滤以仅显示图像
// intent.setType("image/*");
// 要搜索通过已安装的存储提供商提供的所有文档
intent.setType("*/*");
// 如果需要多选，对应的onActivityResult获取Uri通过data.getClipData()
// intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true)
startActivityForResult(intent, READ_REQUEST_CODE);
```

在onActivityResult中获取到文件的Uri，如果是发送文件功能的话，还需要文件名和文件类型，然后加上FileInputStream发送出去，所以需要通过Uri获取文件相关信息

```java
@Override
public void onActivityResult(int requestCode, int resultCode,
                                Intent resultData) {

    if (requestCode == READ_REQUEST_CODE && resultCode == Activity.RESULT_OK) {
        Uri uri = null;
        if (resultData != null) {
            uri = resultData.getData();
        }
    }
}
```

比如：
* 图片目录下的图片Uri类似于`content://com.android.providers.media.documents/document/image%3A616260`，
* 音频目录下`content://com.android.providers.media.documents/document/audio%3A417558`，
* 视频目录下`content://com.android.providers.media.documents/document/video%3A616341`，
* 下载目录下`content://com.android.providers.downloads.documents/document/2884`。

根据Uri可以使用ContentResolver查询相关文件的信息，比如ID、MIME_TYPE等等，查询代码可以看下面的FileUtils，在使用getDataColumn之前需要对Uri进行判断，获取文件种类以及id，比如`image%3A616260`，这就是一个Image文件，并且id为`616260`，`%3A`为冒号`:`，然后根据文件类型，使用相应的Uri查询，比如Image对应`contentUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;`，打印出来的结果是`content://media/external/images/media`，然后selection为`_id=?`，selectionArgs为id值。

```java
public static String getDataColumn(Context context, Uri uri, String selection,
                                    String[] selectionArgs) {

    Cursor cursor = null;
    // ContentProvider查询方式，通过uri加上指定的column，类似于查询数据库，uri中包括了provider的全称以及id，
    // 等价于数据库表与id，column等价于列，这样就可以直接取到对应的值；这里查询的是DISPLAY_NAME，一般来说文件名
    // 应该就包含了文件类型，但是在实际使用中，有的文件的DISPLAY_NAME与文件名并不相同，所以需要知道文件类型，
    // 可以使用MIME_TYPE，还有其他种类的信息例如修改时间等等，并不常用（DATA列已被弃用，但可以查询到文件路径）
    final String column = MediaStore.Images.ImageColumns.DISPLAY_NAME;
    final String[] projection = {
            column
    };

    try {
        // 查询方法调用query
        cursor = context.getContentResolver().query(
            uri, projection, selection, selectionArgs,null);
        if (cursor != null && cursor.moveToFirst()) {
            final int column_index = cursor.getColumnIndexOrThrow(column);
            return cursor.getString(column_index);
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (cursor != null)
            cursor.close();
    }
    return null;
}
```

在测试过程中发现，对于SAF中图片、音频、视频三个目录下的文件都可以获取其正确的信息，但是对于下载目录下的文件存在问题。网上的对于`com.android.providers.downloads`的查询Uri是如下三种，但是全部无法查询到任何结果，`public_downloads`报异常`Unknown URI`（Android 6可以查询到），`my_downloads`查询结果为空，`all_downloads`报异常

```log
java.lang.SecurityException: Permission Denial: reading com.android.providers.downloads.provider.DownloadProvider uri content://downloads/all_downloads/2884 from pid=25434, uid=10786 requires android.permission.ACCESS_ALL_DOWNLOADS, or grantUriPermission()
```

```java
final String id = DocumentsContract.getDocumentId(uri);

if (id != null && id.startsWith("raw:")) {
    return id.substring(4);
}
String[] contentUriPrefixesToTry = new String[]{
        "content://downloads/public_downloads",
        "content://downloads/my_downloads",
        "content://downloads/all_downloads"
};
// 在API 29上多了一个新的Uri MediaStore.Downloads.EXTERNAL_CONTENT_URI，打印结果是
// content://media/external/downloads，类似于上面的MediaStore.Images.Media.EXTERNAL_CONTENT_URI，
// 但是还是查询不到任何结果；如果直接拿content://com.android.providers.downloads.documents/document/2884
// 来查询可以查到MIME_TYPE和DISPLAY_NAME，或者通过context.getContentResolver().getType(uri)获取MIME_TYPE
for (String contentUriPrefix : contentUriPrefixesToTry) {
    Uri contentUri = ContentUris.withAppendedId(Uri.parse(contentUriPrefix), Long.valueOf(id));
    try {
        String path = getDataColumn(context, contentUri, null, null);
        if (path != null) {
            return path;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

对于下载目录下的文件获取其文件名以及文件类型的方式就只能通过原始Uri查询了，但是DISPLAY_NAME并不一定与真实文件名相同，然后通过Uri取文件数据，如下所示

```java
// 读取文件中的字符串
private String readTextFromUri(Uri uri) throws IOException {
    StringBuilder stringBuilder = new StringBuilder();
    try (InputStream inputStream =
            getContentResolver().openInputStream(uri);
            BufferedReader reader = new BufferedReader(
            new InputStreamReader(Objects.requireNonNull(inputStream)))) {
        String line;
        while ((line = reader.readLine()) != null) {
            stringBuilder.append(line);
        }
    }
    return stringBuilder.toString();
}
```

或者取数据的FileInputStream，再利用FileInputStream进行其他操作，比如复制文件或者发送

```java
ContentResolver contentResolver = this.getContentResolver();
ParcelFileDescriptor parcelFileDescriptor = null;
try {
    parcelFileDescriptor = contentResolver.openFileDescriptor(uri, "r");
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
if (parcelFileDescriptor == null) {
    return;
}

FileChannel inputChannel = null;
FileChannel outputChannel = null;

long start = System.currentTimeMillis();

// 为了验证是否获取到数据，可以将数据保存到其他位置，比如这里的aabb.rar
String dest = "/storage/emulated/0/Android/data/us.zoom.androidqdemo/files/aabb.rar";

try {
    FileInputStream inputStream = new FileInputStream(
            parcelFileDescriptor.getFileDescriptor());
    inputChannel = inputStream.getChannel();
    outputChannel = new FileOutputStream(dest).getChannel();

    long srcSize = inputChannel.size();

    long size = outputChannel.transferFrom(inputChannel, 0, srcSize);
    if (size == srcSize) {
        return;
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    long end = System.currentTimeMillis();
    Toast.makeText(this, "Time: " + (end - start), Toast.LENGTH_LONG).show();

    try {
        if (inputChannel != null) {
            inputChannel.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    try {
        if (outputChannel != null) {
            outputChannel.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 1.3 MediaStore使用

MediaStore是用于获取或者添加媒体文件（图片、音频、视频）信息的工具，需要配合ContentResolver使用，MediaStore定义列的名称，通过ContentResolver的query和insert方法去查询和添加文件。

```java
/**
 * 获取系统外部存储内所有的图片
 * @param context
 * @return list
 */
private List<ImageInfo> getImageList(Context context) {
    List<ImageInfo> list = new ArrayList();
    ContentResolver contentResolver = context.getContentResolver();
    // 查询图片需要的Uri
    Uri uri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
    String[] projection = null;
    String selection = null;
    String[] selectionArgs = null;
    String sortOrder = null;
    Cursor cursor = contentResolver.query(uri, projection, selection, selectionArgs, sortOrder);
    if (cursor != null) {
        while (cursor.moveToNext()) {
            ImageInfo imageInfo = new ImageInfo();
            imageInfo.id = cursor.getInt(cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID));
            // 组装图片uri
            imageInfo.uri = ContentUris.withAppendedId(uri, imageInfo.id);
            imageInfo.filePath = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
            imageInfo.mimeType = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.MIME_TYPE));
            imageInfo.title = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.TITLE));
            imageInfo.addTime = cursor.getLong(cursor.getColumnIndex(MediaStore.Images.Media.DATE_ADDED));
            list.add(imageInfo);
        }
        cursor.close();
    }
    return list;
}
```

得到uri后可以通过上面提到的方式取数据也可以直接通过`ImageView.setImageURI(imageInfo.uri)`展示出来。如果有加载缩略图的要求，也可以通过Uri获取缩略图的Bitmap，调用loadThumbnail方法，并且可以指定缩略图大小（视频文件也可以加载缩略图），通过query方式查询缩略图的方式在Android Q上基本失效。

```java
context.getContentResolver().loadThumbnail(imageInfo.uri, new Size(50, 50), null)
```

除了查询之外，比较重要的是插入媒体文件的功能，比如在外部存储的应用私有目录下的图片不会显示在SAF框架中，如果需要将其显示在图片目录中，需要将文件另存到公有目录下，一般是Pictures文件夹中。同理对音频、视频文件也是如此，通过insert方法传入的Uri决定文件保存位置。在Android Q上通过Uri将文件保存到Download目录下似乎不太可行，只能依靠DownloadManager直接下载保存到Download下。

```java
private boolean SavePictureFile(Context context, File file) {
    if (file == null) {
        return false;
    }
    // 首先需要获取到源文件的File对象，然后根据File对象的相关信息构造Uri，比如MIME_TYPE和DISPLAY_NAME等等
    Uri uri = insertFileIntoMediaStore(context, file, true);
    // 然后通过FileInputStream的方式将文件拷贝到目的文件中
    return copyFile(context, file, uri);
}

private Uri insertFileIntoMediaStore(Context context, File file, boolean isPicture) {
    ContentValues contentValues = new ContentValues();
    contentValues.put(MediaStore.Video.Media.DISPLAY_NAME, file.getName());
    contentValues.put(MediaStore.Video.Media.MIME_TYPE, isPicture ? "image/jpeg" : "video/mp4");
    if (Build.VERSION.SDK_INT >= 29) {
        contentValues.put(MediaStore.Video.Media.DATE_TAKEN, file.lastModified());
        // Android Q如果不设置RELATIVE_PATH，则默认保存在Pictures文件夹下，可以通过RELATIVE_PATH添加
        // Pictures/MyPictures子文件夹，文件将会保存在MyPictures中
        // contentValues.put(MediaStore.Video.Media.RELATIVE_PATH, Environment.DIRECTORY_PICTURES + File.separator + "MyPictures");
    }

    Uri uri = null;
    try {
        // 通过insert方法得到公有目录下目的文件的Uri
        uri = context.getContentResolver().insert(
                (isPicture ? MediaStore.Images.Media.EXTERNAL_CONTENT_URI : MediaStore.Video.Media.EXTERNAL_CONTENT_URI)
                , contentValues
        );
    } catch (Exception e) {
        e.printStackTrace();
    }
    return uri;
}

// 文件复制的方式同上
private boolean copyFile(Context context, File srcFile, Uri destFile) {
    ContentResolver contentResolver = context.getContentResolver();
    ParcelFileDescriptor parcelFileDescriptor = null;
    try {
        parcelFileDescriptor = contentResolver.openFileDescriptor(destFile, "w");
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    if (parcelFileDescriptor == null) {
        return false;
    }

    FileChannel inputChannel = null;
    FileChannel outputChannel = null;

    try {
        FileInputStream inputStream = new FileInputStream(
                srcFile);
        inputChannel = inputStream.getChannel();
        outputChannel = new FileOutputStream(parcelFileDescriptor.getFileDescriptor()).getChannel();

        long srcSize = inputChannel.size();

        long size = outputChannel.transferFrom(inputChannel, 0, srcSize);
        if (size == srcSize) {
            return true;
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (inputChannel != null) {
                inputChannel.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            if (outputChannel != null) {
                outputChannel.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    return false;
}
```

## 2. Android系统路径API补充说明

### 2.1 内部存储与外部存储

首先是内部存储与外部存储，内部存储类似`/data/user/0/us.zoom.androidqdemo/`，其中`us.zoom.androidqdemo`是包名，可以通过以下几个方法获取到当前应用的内部路径，内部存储一个最主要的特点就是与应用绑定，如果应用卸载了那么内部存储中应用私有目录的所有文件都会被删除，另一个特点就是内部存储无法直接通过手机中的文件管理或者其他名字的系统应用查看（而外部存储可见），内部存储的空间一般较小，需要谨慎使用，外部存储空间很大。

```java
Environment.getDataDirectory().getAbsolutePath() // /data
getFilesDir().getAbsolutePath() // /data/user/0/us.zoom.androidqdemo/files
getCacheDir().getAbsolutePath() // /data/user/0/us.zoom.androidqdemo/cache
getDir("myFile", MODE_PRIVATE).getAbsolutePath() // /data/user/0/us.zoom.androidqdemo/app_myFile
```

外部存储类似`/storage/emulated/0/Android/data/us.zoom.androidqdemo/files/`（外部存储的应用私有目录）、`/storage/emulated/0/Pictures/Screenshots/`（外部存储的公有目录）这样的路径，我们通过文件管理这个系统应用进入的根目录就是`/storage/emulated/0`，在Android Q之前我们会发现这个目录下有非常多的乱七八糟的文件夹，除了`Alarms, Android, DCIM, Download, Movies, Music, Notifications, Pictures, Podcasts, Ringtones`之外，其他文件或者文件夹都是由你安装的其他APP自行生成的，所以Android Q之前的文件系统极其混乱，因此在Android Q之后不允许对外部存储中公有目录随意访问（可能提示权限拒绝），而内部存储以及外部存储的应用私有目录可以直接通过路径访问，因此在Android Q上我们就会发现外部存储的根目录仅有上面我提到的几个文件夹。

```java
Environment.getExternalStorageDirectory().getAbsolutePath() // /storage/emulated/0
Environment.getExternalStoragePublicDirectory("").getAbsolutePath() // /storage/emulated/0
getExternalFilesDir("").getAbsolutePath() // /storage/emulated/0/Android/data/us.zoom.androidqdemo/files
getExternalCacheDir().getAbsolutePath() // /storage/emulated/0/Android/data/us.zoom.androidqdemo/cache
// 如果手机支持SD卡扩展，那么可以通过getExternalFilesDirs("")获取所有的外部存储（手机内置外部存储+SD卡）
```

现在的Android手机一般情况下存储空间都非常大了基本在32GB起步，64GB比较常见，一般用到的都是外部存储，但是还是需要判断外部存储空间是否可用，比如通过`Environment.getExternalStorageState()`判断是否正常挂载，如果不可用那么就需要使用到内部存储。

### 2.2 缓存与其他文件

在内部存储和外部存储的应用私有目录下会发现两个文件夹`files`和`cache`，很显然，`files`用于存储普通数据，`cache`用于存储缓存数据，如何使用这两个目录存储应用的文件就依赖开发人员的选择了，比如如果是应用本身下载的文件但是不想对外公开，那么可以放在`files`中，如果是应用读写文件过程中产生的临时文件可以放在`cache`中，实际开发时需自行设计。

* 清除缓存：我们知道应用程序在运行过程中需要经过很多过程，比如读入程序，计算，输入输出等等，这些过程中肯定会产生很多的数据，它们在内存中，以供程序运行时调用。所以清除缓存清除的是APP运行过程中所产生的临时数据。 

* 清除数据：清除数据才是真正的删除了我们保存在文件中的数据（永久性数据，如果不人为删除的话会一直保存在文件中）例如当我们在设置里面清除了某个应用的数据，那么`/data/user/0/packname/`和`/storage/emulated/0/Android/data/packname/`下的文件里面的数据会全部删除，包括`cache`，`files`，`lib`，`shared_prefs`等等。

## 3. FileUtils相关代码

```java
package us.zoom.androidqdemo;

/*
 * Copyright (C) 2018 OpenIntents.org
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

import android.content.ContentUris;
import android.content.Context;
import android.content.Intent;
import android.database.Cursor;
import android.database.DatabaseUtils;
import android.net.Uri;
import android.os.Build;
import android.os.Environment;
import android.provider.DocumentsContract;
import android.provider.MediaStore;
import android.provider.OpenableColumns;
import android.util.Log;
import android.webkit.MimeTypeMap;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.core.content.FileProvider;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileFilter;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.text.DecimalFormat;
import java.util.Comparator;

public class FileUtils {
    public static final String DOCUMENTS_DIR = "documents";
    // configured android:authorities in AndroidManifest (https://developer.android.com/reference/android/support/v4/content/FileProvider)
    public static final String AUTHORITY =  "YOUR_AUTHORITY.provider";
    public static final String HIDDEN_PREFIX = ".";
    /**
     * TAG for log messages.
     */
    static final String TAG = "FileUtils";
    private static final boolean DEBUG = false; // Set to true to enable logging
    /**
     * File and folder comparator. TODO Expose sorting option method
     */
    public static Comparator<File> sComparator = (f1, f2) -> {
        // Sort alphabetically by lower case, which is much cleaner
        return f1.getName().toLowerCase().compareTo(
                f2.getName().toLowerCase());
    };
    /**
     * File (not directories) filter.
     */
    public static FileFilter sFileFilter = file -> {
        final String fileName = file.getName();
        // Return files only (not directories) and skip hidden files
        return file.isFile() && !fileName.startsWith(HIDDEN_PREFIX);
    };
    /**
     * Folder (directories) filter.
     */
    public static FileFilter sDirFilter = file -> {
        final String fileName = file.getName();
        // Return directories only and skip hidden directories
        return file.isDirectory() && !fileName.startsWith(HIDDEN_PREFIX);
    };

    private FileUtils() {
    } //private constructor to enforce Singleton pattern

    /**
     * Gets the extension of a file name, like ".png" or ".jpg".
     *
     * @param uri
     * @return Extension including the dot("."); "" if there is no extension;
     * null if uri was null.
     */
    public static String getExtension(String uri) {
        if (uri == null) {
            return null;
        }

        int dot = uri.lastIndexOf(".");
        if (dot >= 0) {
            return uri.substring(dot);
        } else {
            // No extension.
            return "";
        }
    }

    /**
     * @return Whether the URI is a local one.
     */
    public static boolean isLocal(String url) {
        return url != null && !url.startsWith("http://") && !url.startsWith("https://");
    }

    /**
     * @return True if Uri is a MediaStore Uri.
     * @author paulburke
     */
    public static boolean isMediaUri(Uri uri) {
        return "media".equalsIgnoreCase(uri.getAuthority());
    }

    /**
     * Convert File into Uri.
     *
     * @param file
     * @return uri
     */
    public static Uri getUri(File file) {
        return (file != null) ? Uri.fromFile(file) : null;
    }

    /**
     * Returns the path only (without file name).
     *
     * @param file
     * @return
     */
    public static File getPathWithoutFilename(File file) {
        if (file != null) {
            if (file.isDirectory()) {
                // no file to be split off. Return everything
                return file;
            } else {
                String filename = file.getName();
                String filepath = file.getAbsolutePath();

                // Construct path without file name.
                String pathwithoutname = filepath.substring(0,
                        filepath.length() - filename.length());
                if (pathwithoutname.endsWith("/")) {
                    pathwithoutname = pathwithoutname.substring(0, pathwithoutname.length() - 1);
                }
                return new File(pathwithoutname);
            }
        }
        return null;
    }

    /**
     * @return The MIME type for the given file.
     */
    public static String getMimeType(File file) {

        String extension = getExtension(file.getName());

        if (extension.length() > 0)
            return MimeTypeMap.getSingleton().getMimeTypeFromExtension(extension.substring(1));

        return "application/octet-stream";
    }

    /**
     * @return The MIME type for the give Uri.
     */
    public static String getMimeType(Context context, Uri uri) {
        File file = new File(getPath(context, uri));
        return getMimeType(file);
    }

      /**
     * @return The MIME type for the give String Uri.
     */
    public static String getMimeType(Context context, String url) {
        String type = context.getContentResolver().getType(Uri.parse(url));
        if (type == null) {
            type = "application/octet-stream";
        }
        return type;
    }
    
    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is local.
     */
    public static boolean isLocalStorageDocument(Uri uri) {
        return AUTHORITY.equals(uri.getAuthority());
    }

    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is ExternalStorageProvider.
     */
    public static boolean isExternalStorageDocument(Uri uri) {
        return "com.android.externalstorage.documents".equals(uri.getAuthority());
    }

    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is DownloadsProvider.
     */
    public static boolean isDownloadsDocument(Uri uri) {
        return "com.android.providers.downloads.documents".equals(uri.getAuthority());
    }

    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is MediaProvider.
     */
    public static boolean isMediaDocument(Uri uri) {
        return "com.android.providers.media.documents".equals(uri.getAuthority());
    }

    /**
     * @param uri The Uri to check.
     * @return Whether the Uri authority is Google Photos.
     */
    public static boolean isGooglePhotosUri(Uri uri) {
        return "com.google.android.apps.photos.content".equals(uri.getAuthority());
    }

    /**
     * Get the value of the data column for this Uri. This is useful for
     * MediaStore Uris, and other file-based ContentProviders.
     *
     * @param context       The context.
     * @param uri           The Uri to query.
     * @param selection     (Optional) Filter used in the query.
     * @param selectionArgs (Optional) Selection arguments used in the query.
     * @return The value of the _data column, which is typically a file path.
     */
    public static String getDataColumn(Context context, Uri uri, String selection,
                                       String[] selectionArgs) {

        Cursor cursor = null;
        final String column = MediaStore.Files.FileColumns.DATA;
        final String[] projection = {
                column
        };

        try {
            cursor = context.getContentResolver().query(uri, projection, selection, selectionArgs,
                    null);
            if (cursor != null && cursor.moveToFirst()) {
                if (DEBUG)
                    DatabaseUtils.dumpCursor(cursor);

                final int column_index = cursor.getColumnIndexOrThrow(column);
                return cursor.getString(column_index);
            }
        }  catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null)
                cursor.close();
        }
        return null;
    }

     /**
     * Get a file path from a Uri. This will get the the path for Storage Access
     * Framework Documents, as well as the _data field for the MediaStore and
     * other file-based ContentProviders.<br>
     * <br>
     * Callers should check whether the path is local before assuming it
     * represents a local file.
     *
     * @param context The context.
     * @param uri     The Uri to query.
     * @see #isLocal(String)
     * @see #getFile(Context, Uri)
     */
    public static String getPath(final Context context, final Uri uri) {
        String absolutePath = getLocalPath(context, uri);
        return absolutePath != null ? absolutePath : uri.toString();
    }
    
    private static String getLocalPath(final Context context, final Uri uri) {

        if (DEBUG)
            Log.d(TAG + " File -",
                    "Authority: " + uri.getAuthority() +
                            ", Fragment: " + uri.getFragment() +
                            ", Port: " + uri.getPort() +
                            ", Query: " + uri.getQuery() +
                            ", Scheme: " + uri.getScheme() +
                            ", Host: " + uri.getHost() +
                            ", Segments: " + uri.getPathSegments().toString()
            );

        final boolean isKitKat = Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT;

        // DocumentProvider
        if (isKitKat && DocumentsContract.isDocumentUri(context, uri)) {
            // LocalStorageProvider
            if (isLocalStorageDocument(uri)) {
                // The path is the id
                return DocumentsContract.getDocumentId(uri);
            }
            // ExternalStorageProvider
            else if (isExternalStorageDocument(uri)) {
                final String docId = DocumentsContract.getDocumentId(uri);
                final String[] split = docId.split(":");
                final String type = split[0];

                if ("primary".equalsIgnoreCase(type)) {
                    return Environment.getExternalStorageDirectory() + "/" + split[1];
                } else if ("home".equalsIgnoreCase(type)) {
                    return Environment.getExternalStorageDirectory() + "/documents/" + split[1];
                }
            }
            // DownloadsProvider
            else if (isDownloadsDocument(uri)) {

                final String id = DocumentsContract.getDocumentId(uri);

                if (id != null && id.startsWith("raw:")) {
                    return id.substring(4);
                }
                // Android Q 似乎无效
                String[] contentUriPrefixesToTry = new String[]{
                        "content://downloads/public_downloads",
                        "content://downloads/my_downloads",
                        "content://downloads/all_downloads"
                };

                for (String contentUriPrefix : contentUriPrefixesToTry) {
                    Uri contentUri = ContentUris.withAppendedId(Uri.parse(contentUriPrefix), Long.valueOf(id));
                    try {
                        String path = getDataColumn(context, contentUri, null, null);
                        if (path != null) {
                            return path;
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }

                // path could not be retrieved using ContentResolver, therefore copy file to accessible cache using streams
                String fileName = getFileName(context, uri);
                File cacheDir = getDocumentCacheDir(context);
                File file = generateFileName(fileName, cacheDir);
                String destinationPath = null;
                if (file != null) {
                    destinationPath = file.getAbsolutePath();
                    saveFileFromUri(context, uri, destinationPath);
                }

                return destinationPath;
            }
            // MediaProvider
            else if (isMediaDocument(uri)) {
                final String docId = DocumentsContract.getDocumentId(uri);
                final String[] split = docId.split(":");
                final String type = split[0];

                Uri contentUri = null;
                if ("image".equals(type)) {
                    contentUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
                } else if ("video".equals(type)) {
                    contentUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
                } else if ("audio".equals(type)) {
                    contentUri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
                }

                final String selection = "_id=?";
                final String[] selectionArgs = new String[]{
                        split[1]
                };

                return getDataColumn(context, contentUri, selection, selectionArgs);
            }
        }
        // MediaStore (and general)
        else if ("content".equalsIgnoreCase(uri.getScheme())) {

            // Return the remote address
            if (isGooglePhotosUri(uri)) {
                return uri.getLastPathSegment();
            }

            return getDataColumn(context, uri, null, null);
        }
        // File
        else if ("file".equalsIgnoreCase(uri.getScheme())) {
            return uri.getPath();
        }

        return null;
    }

    /**
     * Convert Uri into File, if possible.
     *
     * @return file A local file that the Uri was pointing to, or null if the
     * Uri is unsupported or pointed to a remote resource.
     * @author paulburke
     * @see #getPath(Context, Uri)
     */
    public static File getFile(Context context, Uri uri) {
        if (uri != null) {
            String path = getPath(context, uri);
            if (path != null && isLocal(path)) {
                return new File(path);
            }
        }
        return null;
    }

    /**
     * Get the file size in a human-readable string.
     *
     * @param size
     * @return
     * @author paulburke
     */
    public static String getReadableFileSize(int size) {
        final int BYTES_IN_KILOBYTES = 1024;
        final DecimalFormat dec = new DecimalFormat("###.#");
        final String KILOBYTES = " KB";
        final String MEGABYTES = " MB";
        final String GIGABYTES = " GB";
        float fileSize = 0;
        String suffix = KILOBYTES;

        if (size > BYTES_IN_KILOBYTES) {
            fileSize = size / BYTES_IN_KILOBYTES;
            if (fileSize > BYTES_IN_KILOBYTES) {
                fileSize = fileSize / BYTES_IN_KILOBYTES;
                if (fileSize > BYTES_IN_KILOBYTES) {
                    fileSize = fileSize / BYTES_IN_KILOBYTES;
                    suffix = GIGABYTES;
                } else {
                    suffix = MEGABYTES;
                }
            }
        }
        return String.valueOf(dec.format(fileSize) + suffix);
    }

    /**
     * Get the Intent for selecting content to be used in an Intent Chooser.
     *
     * @return The intent for opening a file with Intent.createChooser()
     */
    public static Intent createGetContentIntent() {
        // Implicitly allow the user to select a particular kind of data
        final Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        // The MIME data type filter
        intent.setType("*/*");
        // Only return URIs that can be opened with ContentResolver
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        return intent;
    }


    /**
     * Creates View intent for given file
     *
     * @param file
     * @return The intent for viewing file
     */
    public static Intent getViewIntent(Context context, File file) {
        //Uri uri = Uri.fromFile(file);
        Uri uri = FileProvider.getUriForFile(context, AUTHORITY, file);
        Intent intent = new Intent(Intent.ACTION_VIEW);
        String url = file.toString();
        if (url.contains(".doc") || url.contains(".docx")) {
            // Word document
            intent.setDataAndType(uri, "application/msword");
        } else if (url.contains(".pdf")) {
            // PDF file
            intent.setDataAndType(uri, "application/pdf");
        } else if (url.contains(".ppt") || url.contains(".pptx")) {
            // Powerpoint file
            intent.setDataAndType(uri, "application/vnd.ms-powerpoint");
        } else if (url.contains(".xls") || url.contains(".xlsx")) {
            // Excel file
            intent.setDataAndType(uri, "application/vnd.ms-excel");
        } else if (url.contains(".zip") || url.contains(".rar")) {
            // WAV audio file
            intent.setDataAndType(uri, "application/x-wav");
        } else if (url.contains(".rtf")) {
            // RTF file
            intent.setDataAndType(uri, "application/rtf");
        } else if (url.contains(".wav") || url.contains(".mp3")) {
            // WAV audio file
            intent.setDataAndType(uri, "audio/x-wav");
        } else if (url.contains(".gif")) {
            // GIF file
            intent.setDataAndType(uri, "image/gif");
        } else if (url.contains(".jpg") || url.contains(".jpeg") || url.contains(".png")) {
            // JPG file
            intent.setDataAndType(uri, "image/jpeg");
        } else if (url.contains(".txt")) {
            // Text file
            intent.setDataAndType(uri, "text/plain");
        } else if (url.contains(".3gp") || url.contains(".mpg") || url.contains(".mpeg") ||
                url.contains(".mpe") || url.contains(".mp4") || url.contains(".avi")) {
            // Video files
            intent.setDataAndType(uri, "video/*");
        } else {
            intent.setDataAndType(uri, "*/*");
        }

        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
        return intent;
    }

    public static File getDownloadsDir() {
        return Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS);
    }

    public static File getDocumentCacheDir(@NonNull Context context) {
        File dir = new File(context.getCacheDir(), DOCUMENTS_DIR);
        if (!dir.exists()) {
            dir.mkdirs();
        }
        logDir(context.getCacheDir());
        logDir(dir);

        return dir;
    }

    private static void logDir(File dir) {
        if(!DEBUG) return;
        Log.d(TAG, "Dir=" + dir);
        File[] files = dir.listFiles();
        for (File file : files) {
            Log.d(TAG, "File=" + file.getPath());
        }
    }

    @Nullable
    public static File generateFileName(@Nullable String name, File directory) {
        if (name == null) {
            return null;
        }

        File file = new File(directory, name);

        if (file.exists()) {
            String fileName = name;
            String extension = "";
            int dotIndex = name.lastIndexOf('.');
            if (dotIndex > 0) {
                fileName = name.substring(0, dotIndex);
                extension = name.substring(dotIndex);
            }

            int index = 0;

            while (file.exists()) {
                index++;
                name = fileName + '(' + index + ')' + extension;
                file = new File(directory, name);
            }
        }

        try {
            if (!file.createNewFile()) {
                return null;
            }
        } catch (IOException e) {
            Log.w(TAG, e);
            return null;
        }

        logDir(directory);

        return file;
    }

//    /**
//     * Writes response body to disk
//     *
//     * @param body ResponseBody
//     * @param path file path
//     * @return File
//     */
//    public static File writeResponseBodyToDisk(ResponseBody body, String path) {
//        try {
//            File target = new File(path);
//
//            InputStream inputStream = null;
//            OutputStream outputStream = null;
//
//            try {
//                byte[] fileReader = new byte[4096];
//
//                inputStream = body.byteStream();
//                outputStream = new FileOutputStream(target);
//
//                while (true) {
//                    int read = inputStream.read(fileReader);
//
//                    if (read == -1) {
//                        break;
//                    }
//
//                    outputStream.write(fileReader, 0, read);
//                }
//
//                outputStream.flush();
//
//                return target;
//            } catch (IOException e) {
//                return null;
//            } finally {
//                if (inputStream != null) {
//                    inputStream.close();
//                }
//
//                if (outputStream != null) {
//                    outputStream.close();
//                }
//            }
//        } catch (IOException e) {
//            return null;
//        }
//    }

    private static void saveFileFromUri(Context context, Uri uri, String destinationPath) {
        InputStream is = null;
        BufferedOutputStream bos = null;
        try {
            is = context.getContentResolver().openInputStream(uri);
            bos = new BufferedOutputStream(new FileOutputStream(destinationPath, false));
            byte[] buf = new byte[1024];
            is.read(buf);
            do {
                bos.write(buf);
            } while (is.read(buf) != -1);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (is != null) is.close();
                if (bos != null) bos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public static byte[] readBytesFromFile(String filePath) {

        FileInputStream fileInputStream = null;
        byte[] bytesArray = null;

        try {

            File file = new File(filePath);
            bytesArray = new byte[(int) file.length()];

            //read file into bytes[]
            fileInputStream = new FileInputStream(file);
            fileInputStream.read(bytesArray);

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }

        return bytesArray;

    }

    public static File createTempImageFile(Context context, String fileName) throws IOException {
        // Create an image file name
        File storageDir = new File(context.getCacheDir(), DOCUMENTS_DIR);
        return File.createTempFile(fileName, ".jpg", storageDir);
    }

    public static String getFileName(@NonNull Context context, Uri uri) {
        String mimeType = context.getContentResolver().getType(uri);
        String filename = null;

        if (mimeType == null && context != null) {
            String path = getPath(context, uri);
            if (path == null) {
                filename = getName(uri.toString());
            } else {
                File file = new File(path);
                filename = file.getName();
            }
        } else {
            Cursor returnCursor = context.getContentResolver().query(uri, null,
                    null, null, null);
            if (returnCursor != null) {
                int nameIndex = returnCursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
                returnCursor.moveToFirst();
                filename = returnCursor.getString(nameIndex);
                returnCursor.close();
            }
        }

        return filename;
    }

    public static String getName(String filename) {
        if (filename == null) {
            return null;
        }
        int index = filename.lastIndexOf('/');
        return filename.substring(index + 1);
    }
}
```

## 参考：

1. [Android Q 隐私权变更：分区存储](https://developer.android.com/preview/privacy/scoped-storage)
2. [Save a file on external storage](https://developer.android.com/training/data-storage/files/external)
3. [Android 10(Android Q) 适配](https://juejin.im/post/5d838a7af265da03ee6a90cd)
4. [使用存储访问框架打开文件](https://developer.android.com/guide/topics/providers/document-provider?hl=zh-cn)
5. [Android Q 沙箱适配多媒体文件总结](https://segmentfault.com/a/1190000019224425)