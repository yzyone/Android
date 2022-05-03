# 【Android开发】如何快速知道某个so来源于哪个库 #

有时候需要查找某些 so 文件是来源于哪个库的，方便删除。以下是一个小技巧：

 

 

在 app 模块的 build.gradle 中，追加以下代码：

```
//列出所有包含有so文件的库信息
tasks.whenTaskAdded { task ->
    if (task.name=='mergeDebugNativeLibs') { //如果是有多个flavor，则用 mergeFlavorDebugNativeLibs的形式
        task.doFirst {
            println("------------------- find so files start -------------------")
            println("------------------- find so files start -------------------")
            println("------------------- find so files start -------------------")
 
            it.inputs.files.each { file ->
                printDir(new File(file.absolutePath))
            }
 
            println("------------------- find so files end -------------------")
            println("------------------- find so files end -------------------")
            println("------------------- find so files end -------------------")
        }
    }
}
 
def printDir(File file) {
    if (file != null) {
        if (file.isDirectory()) {
            file.listFiles().each {
                printDir(it)
            }
        } else if (file.absolutePath.endsWith(".so")) {
            println "find so file: $file.absolutePath"
        }
    }
}
```

然后执行 "gradlew assembleDebug"命令，看到以下输出：

 
```
> Task :app:mergeDebugNativeLibs
------------------- find so files start -------------------
------------------- find so files start -------------------
------------------- find so files start -------------------
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\290908405aa37ee091b2a987a24aa9d0\jetified-animated-gif-2.0.0\jni\arm64-v8a\libgifimage.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\290908405aa37ee091b2a987a24aa9d0\jetified-animated-gif-2.0.0\jni\armeabi-v7a\libgifimage.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\290908405aa37ee091b2a987a24aa9d0\jetified-animated-gif-2.0.0\jni\x86\libgifimage.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\290908405aa37ee091b2a987a24aa9d0\jetified-animated-gif-2.0.0\jni\x86_64\libgifimage.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b2adcba2625cf2da1029c207c31ac6ca\jetified-webpsupport-2.0.0\jni\arm64-v8a\libstatic-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b2adcba2625cf2da1029c207c31ac6ca\jetified-webpsupport-2.0.0\jni\armeabi-v7a\libstatic-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b2adcba2625cf2da1029c207c31ac6ca\jetified-webpsupport-2.0.0\jni\x86\libstatic-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b2adcba2625cf2da1029c207c31ac6ca\jetified-webpsupport-2.0.0\jni\x86_64\libstatic-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\da48f63f1984c1712902b6f0ec8a5e47\jetified-klog-2.2.10-gradle-564\jni\arm64-v8a\libyylog.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\da48f63f1984c1712902b6f0ec8a5e47\jetified-klog-2.2.10-gradle-564\jni\armeabi-v7a\libyylog.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\aa7e29876f055d50ea8fd93c07422010\jetified-flowimagesdk-3.1.2\jni\arm64-v8a\libflowimagesdk.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\aa7e29876f055d50ea8fd93c07422010\jetified-flowimagesdk-3.1.2\jni\armeabi\libflowimagesdk.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\aa7e29876f055d50ea8fd93c07422010\jetified-flowimagesdk-3.1.2\jni\armeabi-v7a\libflowimagesdk.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\aa7e29876f055d50ea8fd93c07422010\jetified-flowimagesdk-3.1.2\jni\x86\libflowimagesdk.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\5c21cdcf9d283b8974fcbe9e04ef19d9\jetified-venus_android-1.8.3.6.1\jni\arm64-v8a\libvenusjni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\5c21cdcf9d283b8974fcbe9e04ef19d9\jetified-venus_android-1.8.3.6.1\jni\armeabi-v7a\libvenusjni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\03b03f82a2d8757132040c3d37bc203c\jetified-androiditna-1.4.0\jni\armeabi\libandroiditna.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\03b03f82a2d8757132040c3d37bc203c\jetified-androiditna-1.4.0\jni\armeabi-v7a\libandroiditna.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\03b03f82a2d8757132040c3d37bc203c\jetified-androiditna-1.4.0\jni\x86\libandroiditna.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d17888a57de2df2c0f1c971aaa4bbdc0\jetified-davincinative-1.3.3-dev2\jni\arm64-v8a\libdavinci_jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d17888a57de2df2c0f1c971aaa4bbdc0\jetified-davincinative-1.3.3-dev2\jni\armeabi\libdavinci_jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d17888a57de2df2c0f1c971aaa4bbdc0\jetified-davincinative-1.3.3-dev2\jni\armeabi-v7a\libdavinci_jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d5ae77453b08a28d847bad415531ce9c\jetified-orangefilter_android-4.7.6\jni\arm64-v8a\libgnustl_shared.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d5ae77453b08a28d847bad415531ce9c\jetified-orangefilter_android-4.7.6\jni\arm64-v8a\liborangefilterjni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d5ae77453b08a28d847bad415531ce9c\jetified-orangefilter_android-4.7.6\jni\armeabi-v7a\libgnustl_shared.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d5ae77453b08a28d847bad415531ce9c\jetified-orangefilter_android-4.7.6\jni\armeabi-v7a\liborangefilterjni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\8ad36380beae21061f89538289417f11\jetified-nativeimagefilters-2.0.0\jni\arm64-v8a\libnative-filters.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\8ad36380beae21061f89538289417f11\jetified-nativeimagefilters-2.0.0\jni\armeabi-v7a\libnative-filters.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\8ad36380beae21061f89538289417f11\jetified-nativeimagefilters-2.0.0\jni\x86\libnative-filters.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\8ad36380beae21061f89538289417f11\jetified-nativeimagefilters-2.0.0\jni\x86_64\libnative-filters.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\0a06bd3d567873fed34b6f5363b21028\jetified-imagepipeline-2.0.0\jni\arm64-v8a\libimagepipeline.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\0a06bd3d567873fed34b6f5363b21028\jetified-imagepipeline-2.0.0\jni\armeabi-v7a\libimagepipeline.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\0a06bd3d567873fed34b6f5363b21028\jetified-imagepipeline-2.0.0\jni\x86\libimagepipeline.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\0a06bd3d567873fed34b6f5363b21028\jetified-imagepipeline-2.0.0\jni\x86_64\libimagepipeline.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\5c8495a7d95c6c663a3537053d39bcf4\jetified-nativeimagetranscoder-2.0.0\jni\arm64-v8a\libnative-imagetranscoder.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\5c8495a7d95c6c663a3537053d39bcf4\jetified-nativeimagetranscoder-2.0.0\jni\armeabi-v7a\libnative-imagetranscoder.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\5c8495a7d95c6c663a3537053d39bcf4\jetified-nativeimagetranscoder-2.0.0\jni\x86\libnative-imagetranscoder.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\5c8495a7d95c6c663a3537053d39bcf4\jetified-nativeimagetranscoder-2.0.0\jni\x86_64\libnative-imagetranscoder.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\arm64-v8a\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\arm64-v8a\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\armeabi\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\armeabi\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\armeabi-v7a\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\armeabi-v7a\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\mips\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\mips\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\mips64\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\mips64\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\x86\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\x86\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\x86_64\libcocklogic-1.1.3.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\99ec12ed64d22b75f9c1f4df9f386248\jetified-umeng-push-6.0.1\jni\x86_64\libtnet-3.1.14.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ce67796daa0f43f32ff240d63ea0adf0\jetified-webpdecoder-1.6.4.10.0\jni\arm64-v8a\libglide-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ce67796daa0f43f32ff240d63ea0adf0\jetified-webpdecoder-1.6.4.10.0\jni\armeabi\libglide-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ce67796daa0f43f32ff240d63ea0adf0\jetified-webpdecoder-1.6.4.10.0\jni\armeabi-v7a\libglide-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ce67796daa0f43f32ff240d63ea0adf0\jetified-webpdecoder-1.6.4.10.0\jni\x86\libglide-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ce67796daa0f43f32ff240d63ea0adf0\jetified-webpdecoder-1.6.4.10.0\jni\x86_64\libglide-webp.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d34f23183543af786958a133d10f5677\jetified-crashreport-2.2.15\jni\arm64-v8a\libyycrashreport.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\d34f23183543af786958a133d10f5677\jetified-crashreport-2.2.15\jni\armeabi-v7a\libyycrashreport.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ee6cc6d4c69ee5cc189a11096fb868de\jetified-cronet_lib-1.4.0\jni\arm64-v8a\libcronet.72.0.3602.0.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\ee6cc6d4c69ee5cc189a11096fb868de\jetified-cronet_lib-1.4.0\jni\armeabi-v7a\libcronet.72.0.3602.0.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\58d275cc28eaff2429d95cdd525bfa45\jetified-objectbox-android-2.3.4\jni\arm64-v8a\libobjectbox-jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\58d275cc28eaff2429d95cdd525bfa45\jetified-objectbox-android-2.3.4\jni\armeabi-v7a\libobjectbox-jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\58d275cc28eaff2429d95cdd525bfa45\jetified-objectbox-android-2.3.4\jni\x86\libobjectbox-jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\58d275cc28eaff2429d95cdd525bfa45\jetified-objectbox-android-2.3.4\jni\x86_64\libobjectbox-jni.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\arm64-v8a\libnms.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\arm64-v8a\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\armeabi\libnms.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\armeabi\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\armeabi-v7a\libnms.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\armeabi-v7a\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\mips\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\mips64\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\x86\libnms.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\x86\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\x86_64\libnms.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\x86_64\libtobEmbedEncrypt.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\arm64-v8a\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\armeabi\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\armeabi-v7a\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\mips\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\mips64\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\x86\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\e6a8b2249ba7c0cef42eb10d9e0d57b2\jetified-android-gif-drawable-1.2.6\jni\x86_64\libpl_droidsonroids_gif.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\f74a363bd3e4870c9874b0661e1b21d6\jetified-ffmpeg-neon-2.9.4\jni\arm64-v8a\libffmpeg-neon.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\f74a363bd3e4870c9874b0661e1b21d6\jetified-ffmpeg-neon-2.9.4\jni\armeabi-v7a\libffmpeg-neon.so
find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\f74a363bd3e4870c9874b0661e1b21d6\jetified-ffmpeg-neon-2.9.4\jni\x86\libffmpeg-neon.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libBaiduMapSDK_base_v5_1_0.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libBaiduMapSDK_map_v5_1_0.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreanw.10.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreanw.13.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreanw.14.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreanw.18.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreanw.21.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreiomx.10.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreiomx.13.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcoreiomx.14.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libcorejni.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\libhiidostatisjni.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi\liblocSDK7b.so
find so file: C:\Workspace\bi\bi\app\build\intermediates\merged_jni_libs\biDebug\out\armeabi-v7a\libhiidostatisjni.so
find so file: C:\Workspace\bi\bi\bi-videoplayer\build\intermediates\library_jni\debug\jni\armeabi\libksyplayer.so
find so file: C:\Workspace\bi\bi\bi-videoplayer\build\intermediates\library_jni\debug\jni\armeabi-v7a\libksyplayer.so
find so file: C:\Workspace\bi\bi\bi-push\build\intermediates\library_jni\debug\jni\armeabi\libbdpush_V2_9.so
find so file: C:\Workspace\bi\bi\bi-push\build\intermediates\library_jni\debug\jni\armeabi-v7a\libbdpush_V2_9.so
------------------- find so files end -------------------
------------------- find so files end -------------------
------------------- find so files end -------------------
```

例如：libnms.so，它出现在以下目录：

	find so file: D:\Gradle_Repo\caches\transforms-2\files-2.1\b4fcab3e51647966a62c76a35ef04aa5\jetified-open_ad_sdk\jni\arm64-v8a\libnms.so

就是说由 open_ad_sdk.aar 引入的。

————————————————

版权声明：本文为CSDN博主「又吹风_Bassy」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/eieihihi/article/details/109289312