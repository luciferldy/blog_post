+++
date = "2015-05-19T17:04:39+08:00"
menu = "main"
tags = ["android"]
title = "Android MediaStore 的基础知识"

+++

##### 前言

一直总想做一个音乐播放器，因此在网上也搜寻了很多资料，大约几个月前，谷歌开源了一个音乐播放器
[Android-UniversalMusicPlayer](https://github.com/googlesamples/android-UniversalMusicPlayer)，个人感觉不错，只不过需要 API 21 以及最新的 build tools…… 于是在自己的机器上木有成功运行。但是代码还是有很大的观赏性的，现在安卓市场上的播放器，很多人都比较推崇 **网易云音乐** ，我个人感觉 [**谷歌音乐播放器**](http://www.coolapk.com/apk/com.google.android.music) 也是一个体验很强的本地音乐播放器（网络服务需要google service）。但是谷歌音乐播放器和Android-Universal-UniversalMusicPlayer从截图来看有点大同小异，大家可以试一下。

##### Android MeidaStore

在 Android 上进行音乐播放器的开发有两点的方便之处在于不需要自行扫描音乐文件和创建音乐文件的数据库。 

每当 Android 启动或者新的SD卡插入时，MusicSanner 服务会在后台自动扫描 SD 卡上的文件资源，将 SD 卡上的音乐媒体信息加入到 MediaStore 数据库中，程序可以直接从 MediaStore 中读取相应的媒体信息。当然也也可以调用 MusicScanner 主动进行扫描，因为当你在网络上下载一首歌曲后，系统不会自动进行扫描。

打开 DDMS，在 data/data/com.android.provider.media 下找到 databases 文件夹，下面的 external.db 代表在外存扫描的结果，internal.db 代表在内存的扫描结果。在 external.db 里面的 audio, albums, artists, audio_genres 表分别存储的是歌曲，专辑，艺术家，流派的信息。（说到这里，给大家推荐一个sqlite的使用工具[sqlite expert](http://www.sqliteexpert.com/) 专业版需要收费，个人版不需要收费）

这段代码是我调用系统的resolver,获得专辑信息的代码

	ArrayList<HashMap<String, String>> albums = new ArrayList<>();
    ContentResolver resolver = context.getContentResolver();
    try {
        Cursor cursor = resolver.query(MediaStore.Audio.Albums.EXTERNAL_CONTENT_URI, null, null, null,
                MediaStore.Audio.Albums.DEFAULT_SORT_ORDER);
        HashMap<String, String> item;

	//            Log.d(TAG_MSG, "albums id");
        while (cursor.moveToNext()){
            item = new HashMap<>();

	//                Log.d(TAG_MSG, cursor.getString(cursor.getColumnIndex(MediaStore.Audio.Albums._ID)));

            item.put(albumId, cursor.getString(cursor.getColumnIndex(MediaStore.Audio.Albums._ID)));
            item.put(albumName, cursor.getString(cursor.getColumnIndex(MediaStore.Audio.Albums.ALBUM)));
            item.put(artistName, cursor.getString(cursor.getColumnIndex(MediaStore.Audio.Albums.ARTIST)));
            item.put(songNumber, cursor.getString(cursor.getColumnIndex(MediaStore.Audio.Albums.NUMBER_OF_SONGS)));
            albums.add(item);
        }
        return albums;
    }catch (Exception e){
        e.printStackTrace();
        return albums;
    }

MediaStore.Audio.Albums.\_ID 和 Meida.Audio.Meida.ALBUM\_ID是对应的

##### 疑问

没有读过一些更加底层的代码，所以对它的实现机理还不是很懂

* 在 albums 表中木有歌曲数量这一属性
* 当我试图调用 MediaStore.Audio.Album.ALBUM_ID 时，系统会报错。 

希望能够有人能够解答，谢谢。