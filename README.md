# RxAndroidAudio
Android Audio encapsulation library, with part Rx support.

[ ![Download](https://api.bintray.com/packages/piasy/maven/RxAndroidAudio/images/download.svg) ](https://bintray.com/piasy/maven/RxAndroidAudio/_latestVersion)

## Usage

### Add to gradle dependency of your module build.gradle:

```gradle
allprojects {
    repositories {
        maven {
            url  "http://dl.bintray.com/piasy/maven"
        }
    }
}

dependencies {
    compile 'com.github.piasy:rxandroidaudio:1.4.0'
    compile 'com.github.piasy:AudioProcessor:1.4.0'
}
```

### Use in code:

#### Record to file:

```java
mAudioRecorder = AudioRecorder.getInstance();
mAudioFile = new File(
        Environment.getExternalStorageDirectory().getAbsolutePath() +
                File.separator + System.nanoTime() + ".file.m4a");
mAudioRecorder.prepareRecord(MediaRecorder.AudioSource.MIC,
        MediaRecorder.OutputFormat.MPEG_4, MediaRecorder.AudioEncoder.AAC,
        mAudioFile);
mAudioRecorder.startRecord();
// ...
mAudioRecorder.stopRecord();
```

**Note**: If you record a aac file, the sound quality will be poor if the sample rate and encoding
bit rate is low, the sound quality will increase when you set a bigger sample rate and encoding
bit rate, but as the sound quality improve, the recorded file size will also increase.

#### Play a file
With PlayConfig, to set audio file or audio resource, set volume, or looping:

```java
mRxAudioPlayer.play(PlayConfig.file(audioFile).looping(true).build())
        .subscribeOn(Schedulers.io())
        .subscribe(new Action1<Boolean>() {
            @Override
            public void call(Boolean aBoolean) {
                // play finished
                // NOTE: if looping, the Single will never finish, you need stop playing
                // onDestroy, otherwise, memory leak will happen!
            }
        });
```

#### Full example of PlayConfig

```java
PlayConfig.file(audioFile) // play a local file
    //.res(getApplicationContext(), R.raw.audio_record_end) // or play a raw resource
    .looping(true) // loop or not
    .leftVolume(1.0F) // left volume
    .rightVolume(1.0F) // right volume
    .build(); // build this config and play!
```

#### Record a stream:

```java
mOutputFile = new File(Environment.getExternalStorageDirectory().getAbsolutePath() +
        File.separator + System.nanoTime() + ".stream.m4a");
mOutputFile.createNewFile();
mFileOutputStream = new FileOutputStream(mOutputFile);
mStreamAudioRecorder.start(new StreamAudioRecorder.AudioDataCallback() {
    @Override
    public void onAudioData(byte[] data, int size) {
        if (mFileOutputStream != null) {
            try {
                mFileOutputStream.write(data, 0, size);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void onError() {
        mBtnStart.post(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(getApplicationContext(), "Record fail",
                        Toast.LENGTH_SHORT).show();
                mBtnStart.setText("Start");
                mIsRecording = false;
            }
        });
    }
});
```

#### Play a stream:

```java
Observable.just(mOutputFile).subscribeOn(Schedulers.io()).subscribe(new Action1<File>() {
    @Override
    public void call(File file) {
        try {
            mStreamAudioPlayer.init();
            FileInputStream inputStream = new FileInputStream(file);
            int read;
            while ((read = inputStream.read(mBuffer)) > 0) {
                mStreamAudioPlayer.play(mBuffer, read);
            }
            inputStream.close();
            mStreamAudioPlayer.release();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});
```

#### Change the sound effect in stream mode:

``` java
mStreamAudioPlayer.play(
    mAudioProcessor.process(mRatio, mBuffer, StreamAudioRecorder.DEFAULT_SAMPLE_RATE),
    len);
```

See [full example](https://github.com/Piasy/RxAndroidAudio/tree/master/app) for more details.

[Download demo apk](http://fir.im/RXA).

## Contribution are welcome!
+  Please follow [my code style based on SquareAndroid](https://github.com/Piasy/java-code-styles)
