
快速开始
=

- **现在以模式二为例开始我们的Demo。  
目前SDK是以aar形式提供，所以需要使用Android Studio开发。把"ratn-release-xx-online.aar"拷贝到Libs文件夹下。在muoudle的build.gradle文件中添加。**

``` gradle
repositories {
    flatDir { dirs 'libs' }
}

dependencies {
    compile(name:'ratn-release-1.0-online',ext:'aar')
}
```
- **我们需要你创建一个带有按钮的页面，就像这样**  

![image.png](https://upload-images.jianshu.io/upload_images/11080649-b5a3d5be07ea6582.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **然后我们需要初始化我们的VUI**  
```Java
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mode_vad);
        initSDK();
        initView();
    }
``` 
- **initSDK()
初始化之前，需要先初始化参数。像这样**  
```Java
    private void initSDK(){
        VUIApi.InitParam.InitParamBuilder builder = new VUIApi.InitParam.InitParamBuilder();

        boolean isUseOnlineTTS = true;//设置TTS是否使用在线模式
        if (isUseOnlineTTS) {
            ttsType = RTTSPlayer.TTSType.TYPE_ONLINE;
        } else {
            ttsType = RTTSPlayer.TTSType.TYPE_OFFLINE;
        }

        builder.setLanguage("cmn-CHN")
                .setTTSType(ttsType)
                .setVUIType(VUIApi.VUIType.AUTO)
                .setAudioGenerator(new CustomAndroidAudioGenerator());

        mVUIApi = VUIApi.getInstance();
        mVUIApi.init(this, builder.build(), mInitListener);//绑定初始化监听器
    }
```
- **接下来**  
```Java
   private void initView() {
        mASRResultText = (TextView) findViewById(R.id.result_vad_mode);
        mStartBtn = (Button) findViewById(R.id.btn_start_vad_mode);
        mStartBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!isRecording) {
                    isRecording = true;
                    mStartBtn.setText("点击结束识别");
                    //开始识别
                    AutoTypeController controller = (AutoTypeController) mVUIApi.startRecognize();
                    controller.manualWakeup();
                } else {
                    mStartBtn.setText("点击开始识别");
                    isRecording = false;
                    //结束识别
                    mVUIApi.stopRecognize();
                }
            }
        });
    }
```
- **重要的监听器  
InitListener  初始化回调**  
```Java 
    InitListener mInitListener = new InitListener() {

        @Override
        public void onSuccess() {
        }

        @Override
        public void onFail(RError message) {
        }
    };
```
- **RASRListener 识别结果回调**  
```Java
    RASRListener mASRListener = new RASRListener() {

        @Override
        public void onASRResult(ASRResult result) {
            String restText = "[识别结果]:" + result.getResultText();
        }

        @Override
        public void onFail(RError message) {
        }

        @Override
        public void onWakeUp(String json) {
        }

        @Override
        public void onEvent(EventType event) {
        }
    };
```
- **OnAIResponseListener  AI语义结果回调**  
```Java
    OnAIResponseListener mAIResponseListener = new OnAIResponseListener() {

        @Override
        public void onResult(String AI_JSON) {
        }

        @Override
        public void onFail(RError message) {
        }
    };
```
- **现在就可以运行你的App体验效果了。Have Fun！:blush:**  
