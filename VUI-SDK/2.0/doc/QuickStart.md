
快速开始
=

- **现在以模式二 VAD+ASR+TTS 为例开始我们的Demo。  
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
*注：deveceID需要您来告诉我。可以查看如何[申请SN号](https://github.com/271766152/docs/blob/master/VUI-SDK/2.0/doc/%E8%B4%A6%E5%8F%B7%E7%94%B3%E8%AF%B7%E6%96%B9%E6%B3%95.md)。*

![image.png](https://github.com/271766152/docs/blob/master/VUI-SDK/2.0/doc/img/demo-2.png)

- **先从onCreate开始**  
```Java
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mode_vad);

        initView();
        initVUIParam(getIntent().getStringExtra(DemoMainActivity.DEVICE_ID));
    }
``` 

- **接下来**  
```Java
   private void initView() {
        asrResultText = (TextView) findViewById(R.id.result_vad_mode);
        btnStart = (Button) findViewById(R.id.btn_start_vad_mode);
        btnStart.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (!isRecording) {
                    isRecording = true;
                    btnStart.setText("点击结束识别");
                    //开始识别
                    AutoTypeController controller = (AutoTypeController) vuiApi.startRecognize();
                    //开始识别后，将控制器设置成已经唤醒的状态
                    if (controller instanceof AutoTypeController) {
                        ((AutoTypeController) controller).manualWakeup();
                    }
                } else {
                    btnStart.setText("点击开始识别");
                    isRecording = false;
                    //结束识别
                    vuiApi.stopRecognize();
                }
            }
        });
    }
```

- **initVUIParam(String deviceID)
初始化之前，需要先初始化参数。像这样**  
```Java
    private void initVUIParam(String deviceID) {
        Log.d(TAG, "deviceID= " + deviceID);
        UserInfo userInfo = new UserInfo();
        userInfo.setDeviceID(deviceID); //必须设置此字段

        VUIApi.InitParam.InitParamBuilder builder = new VUIApi.InitParam.InitParamBuilder();

        builder.setUserInfo(userInfo)//设置用户信息，必须设置
                .setAudioGenerator(new CustomAndroidAudioGenerator())//设置音频源
                .addOfflineFileName("test_offline")//设置离线词文件
                .setTTSType(RTTSPlayer.TTSType.TYPE_ONLINE)//设置语音合成方式,默认是离线
                .setVUIType(VUIApi.VUIType.AUTO);//设置交互方式，AUTO（唤醒后自动开启cloud识别，说唤醒词开始，包含VAD, 直到手动停止）

        vuiApi = VUIApi.getInstance();
        vuiApi.init(this, builder.build(), initListener);//绑定初始化监听器
    }
```

- **重要的监听器  
InitListener  初始化回调**  
```Java 
    InitListener initListener = new InitListener() {
        @Override
        public void onSuccess() {
            btnStart.setClickable(true);
            handler.obtainMessage(MSG_TTS_PLAYER, "初始化成功，现在可以使用识别功能了").sendToTarget();
            vuiApi.setASRListener(asrListener);//绑定ASR监听器,没有AI结果；
            vuiApi.setOnAIResponseListener(aiResponseListener); //绑定AI监听器；如果不需要AI结果，可以不设置
            reprotLocation(); //上报wifi信息，用于识别中用到位置信息

        }

        @Override
        public void onFail(RError message) {
            Toast.makeText(
                VADModeActivity.this, 
                "初始化失败！！！message = " + message.getFailDetail(),
                Toast.LENGTH_SHORT).show();
        }
    };
```

- **reprotLocation() 不用担心，我们只是用它来得到您的大体位置，然后我们的AI才能知道您想干什么。比如：您说今天天气怎么样？**
```Java
    /**
     * reprotLocation wifi信息，只有上报了wifi信息，当用户查询跟位置相关的信息时才会返回结果，比如：今天的天气怎么样
     */
    private void reprotLocation() {
        WifiManager wifiManager = (WifiManager) getApplication().getApplicationContext()
            .getSystemService(Context.WIFI_SERVICE);
        vuiApi.reportLocationInfo(wifiManager.getScanResults());
    }
```

- **RASRListener 识别结果回调**  
```Java
   //设置ASR回调接口。ASR返回的结果都在此接口中回调,不带AI。
    RASRListener asrListener = new RASRListener() {
        @Override
        public void onASRResult(ASRResult result) {
            //如果是需要带AI的结果，此回调结果可以不做处理；
            Log.d(TAG, "ASRResult " + (result.getResultType() == ASRResult.TYPE_OFFLINE ? "offline " : " online ") + " text " + result.getResultText());
//                handler.obtainMessage(MSG_SHOW_RESULT, result.getResultText()).sendToTarget();
        }

        @Override
        public void onFail(RError message) {
            Log.e(TAG, "asr error: " + message.getFailDetail());
        }

        @Override
        public void onWakeUp(String json) {
            Log.e(TAG, "asr wakeup: " + json);
        }

        @Override
        public void onEvent(EventType event) {
            Log.e(TAG, "asr onEvent: " + event.toString());
        }
    };
```
- **OnAIResponseListener  AI语义结果回调**  
```Java
    //设置AI回调接口。AI返回的结果都在此接口中回调，如果不需要AI结果，可以不设置此回调接口。
    OnAIResponseListener aiResponseListener = new OnAIResponseListener() {
        @Override
        public void onResult(String json) {
            Log.e(TAG, "ai json: " + json);
            handler.obtainMessage(MSG_SHOW_RESULT, json).sendToTarget();
        }

        @Override
        public void onFail(RError message) {
            Log.e(TAG, "ai fail: " + message.getFailDetail());
        }
    };
```
- **当然还有我们的TTS**
```Java
/**
     * ttsSpeak 播放tts
     *
     * @param message tts播放的内容
     */
    private void ttsSpeak(String message) {
        vuiApi.speak(message, new RTTSListener() {

            @Override
            public void onSpeakBegin() {
                Log.d(TAG, "onSpeakBegin");
                Toast.makeText(getApplicationContext(), "speak", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onCompleted() {
                Log.d(TAG, "onCompleted");
                handler.obtainMessage(MSG_INIT_COMPLETE).sendToTarget();
            }

            @Override
            public void onError(int code) {
                Log.e(TAG, "onError " + code);
            }
        });
    }
```

- **我们的handler**
```Java
    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) {
                case MSG_SHOW_RESULT:
                    asrResultText.setText((String) msg.obj);
                    break;
                case MSG_TTS_PLAYER:
                    ttsSpeak((String) msg.obj);
                    break;
                case MSG_INIT_COMPLETE:
                    btnStart.setText(getString(R.string.start_vad_mode));
                    btnStart.setEnabled(true);
                    break;
                default:
                    break;
            }
        }
    };
```


- **现在就可以运行你的App体验效果了。Have Fun！:blush:**  
