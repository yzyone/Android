
# Android AudioRecord录制PCM以及转换为wav和mp3

--

### 1.录制pcm

**pcm介绍**

pcm是指音频裸数据是脉冲编码调制数据。描述一段PCM数据通常以下几个概念：

- 量化格式（SampleFormat）又叫位深度：表示可以记录声音的动态范围，代表分贝
- 采样率（SampleRate）：可以表示声音频率范围，通过压缩PCM就是根据人耳能听到的频率来的
- 声道数（Channel）：录制或播放时在不同空间位置采集或回放的相互独立的音频信号，也就是对应的音源数量和扬声器数量。

关于如何计算PCM文件的大小（例：采样率44100，位深度16，声道数2，一分钟）：

文件大小=比特率×时长÷8=位深度×采样率×声道数×时间÷8=44100×16×2÷8≈10.34M

**使用AudioRecord进行采集**

第一步：初始化AudioRecord

```
AudioRecord mAudioRecord = new AudioRecord(
    			MediaRecorder.AudioSource.MIC,//音频源
                44100,//采样率
                AudioFormat.CHANNEL_IN_MONO,//单声道，双声道用CHANNEL_IN_STEREO
                //api28以上可以直接录制MP3
                AudioFormat.ENCODING_PCM_16BIT,//位深度为16bit的PCM文件
                mBufferSize //缓冲区大小，可以根据AudioRecord.getMinBufferSize()计算
        );
```

第二步：开始录制，通过全局变量isRecord进行录制控制

```
//启动线程进行pcm录制
class RecordThread implements Runnable {

        @Override
        public void run() {
            mAudioRecord.startRecording();
            FileOutputStream fos = null;
            try {
                Log.i(TAG, "文件地址: " + mPcmFilePath);
                fos = new FileOutputStream(mPcmFilePath);
                byte[] bytes = new byte[mBufferSize];
                while (isRecord) {
                    mAudioRecord.read(bytes, 0, bytes.length);
                    fos.write(bytes, 0, bytes.length);
                    fos.flush();
                }
                Log.i(TAG, "停止录制");
                mAudioRecord.stop();
                fos.flush();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (fos != null) {
                    try {
                        fos.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }

        }
    }
```

第三步.停止录制

```
 public void stopRecord() {
        isRecord = false;
    }
```

以上就是关于pcm的录制，pcm只是音频原始数据，接下来，我们将进行pcm文件转换

### 2.pcm文件转为WAV格式

wav：微软公司专门为Windows开发的一种标准数字音频文件，该文件能记录各种单声道或立体声的声音信息，并能保证声音不失真。简单说，wav格式就是pcm文件加上了头信息。其头文件内容可以参考相关协议。

```
/**
     * 加入wav文件头
     */
    private void writeWaveFileHeader(FileOutputStream out, long totalAudioLen,
                                     long totalDataLen, long longSampleRate, int channels, long byteRate)
            throws IOException {
        byte[] header = new byte[44];
        // RIFF/WAVE header
        header[0] = 'R';
        header[1] = 'I';
        header[2] = 'F';
        header[3] = 'F';
        header[4] = (byte) (totalDataLen & 0xff);
        header[5] = (byte) ((totalDataLen >> 8) & 0xff);
        header[6] = (byte) ((totalDataLen >> 16) & 0xff);
        header[7] = (byte) ((totalDataLen >> 24) & 0xff);
        //WAVE
        header[8] = 'W';
        header[9] = 'A';
        header[10] = 'V';
        header[11] = 'E';
        // 'fmt ' chunk
        header[12] = 'f';
        header[13] = 'm';
        header[14] = 't';
        header[15] = ' ';
        // 4 bytes: size of 'fmt ' chunk
        header[16] = 16;
        header[17] = 0;
        header[18] = 0;
        header[19] = 0;
        // format = 1
        header[20] = 1;
        header[21] = 0;
        header[22] = (byte) channels;
        header[23] = 0;
        header[24] = (byte) (longSampleRate & 0xff);
        header[25] = (byte) ((longSampleRate >> 8) & 0xff);
        header[26] = (byte) ((longSampleRate >> 16) & 0xff);
        header[27] = (byte) ((longSampleRate >> 24) & 0xff);
        header[28] = (byte) (byteRate & 0xff);
        header[29] = (byte) ((byteRate >> 8) & 0xff);
        header[30] = (byte) ((byteRate >> 16) & 0xff);
        header[31] = (byte) ((byteRate >> 24) & 0xff);
        // block align
        header[32] = (byte) (2 * 16 / 8);
        header[33] = 0;
        // bits per sample
        header[34] = 16;
        header[35] = 0;
        //data
        header[36] = 'd';
        header[37] = 'a';
        header[38] = 't';
        header[39] = 'a';
        header[40] = (byte) (totalAudioLen & 0xff);
        header[41] = (byte) ((totalAudioLen >> 8) & 0xff);
        header[42] = (byte) ((totalAudioLen >> 16) & 0xff);
        header[43] = (byte) ((totalAudioLen >> 24) & 0xff);
        out.write(header, 0, 44);
    }
```

### 3.pcm转mp3

mp3：mp3是一种音频压缩技术，利用人耳对高频声音信号不敏感的特性，将时域波形信号转换成频域信号，并划分成多个频段，对不同的频段使用不同的压缩率。pcm转mp3，用的是开源库lame，由于相对篇幅较大，放到下一篇：https://blog.csdn.net/s591628545/article/details/104526112

参考内容：

https://www.jb51.net/article/130934.htm

项目地址：https://github.com/kingdavidsun/AudioSampling.git

————————————————

版权声明：本文为CSDN博主「羽过天惊」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/s591628545/article/details/104525958