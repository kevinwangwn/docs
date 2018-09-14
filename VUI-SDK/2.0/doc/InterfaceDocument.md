

## 一.初始化SDK
#### 1.初始化
```Java
VUIApi.getInstance().init(context, initParam,initListener);
```
#### 2. InitParam 构造方法

```Java
        VUIApi.InitParam.InitParamBuilder builder = new VUIApi.InitParam.InitParamBuilder();

        builder.setUserInfo(userInfo)//设置用户信息，必须设置（userInfo 参数见快速开始）
                .setAudioGenerator(new CustomAndroidAudioGenerator())//设置音频源
                .setTTSType(RTTSPlayer.TTSType.TYPE_ONLINE)//设置语音合成方式,默认是离线,设置为在线
                .setVUIType(VUIApi.VUIType.AUTO);//设置交互方式；手动且单次识别，需要手动开始 和 手动停止
```
                
*注意：以上为构造InitParam必填参数*

#### 3. InitParam 参数列表 

参数 | 说明 | 必填  
------------ | ------------ | ------------ 
setLanguage() | 设置ASR/TTS/AI的语言 | 是
setVUIType | 设置VUI交互方式 | 是
setTTSType | 设置TTS在线/离线模式 |是
setTTSSpeaker | 如果TTS采用离线方式，这里设置是发音人。如果是采用在线方式，这是设置的是TTS语言 |否(默认"Li-Li")
setAudioGenerator() | 设置语音识别的音源,设置为RooboAECRecorder,是带AEC功能 |是
setUserInfo() | 通常不会调用此方法，仅用于给客户预分配SN和PublicKey的场景;如果用户的SN号是预分配的模式,就必须调用此接口设置SN号 | 否 
setTokenType() | 设置token的类型,默认是内部维护token,如果是外部设置token，同时需要设置setDeviceInfo()的信息 | 否 
setDeviceInfo() | 设置SN号和token,同时需要设置setTokenType(VUIApi.TokenType.TYPE_TOKEN_EXTERNAL_SETTING) | 否 
 
## 二.语音识别   
    
#### 1.开始语音识别
```Java
VUIApi.getInstance().startRecognize();
```

#### 2.停止语音识别
```Java
VUIApi.getInstance().stopRecognize();
```

#### 3.暂停语音识别
```Java
VUIApi.getInstance().cancelRecognize();
```

#### 4.语音识别监听器
```Java
VUIApi.getInstance().setASRListener(RASRListener listener)

RASRListener asrListener = new RASRListener() {
        @Override
        public void onASRResult(ASRResult result) {
        		result.getResultText();//通过该方法获取识别出的文本
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
## 三.语义解析
#### 1.AI语义解析
```Java
VUIApi.getInstance().aiQuery(String text);//text为需要进行AI语义解析的文本
```

#### 2.AI语义解析监听器
```Java
VUIApi.getInstance().setOnAIResponseListener(OnAIResponseListener listener)

 OnAIResponseListener aiResponseListener = new OnAIResponseListener() {
        @Override
        public void onResult(String json) {
           //json返回了AI语义解析的结果
        }

        @Override
        public void onFail(RError rError) {
        }
    };
```
#### 3.AI语义解析上下文
```Java
VUIApi.getInstance().setAIContext(String context);
//RooboAI后台的语义理解是支持上下文的，开发者可以设置上下文，就支持多轮对话。
//在AIResponseListener回调的中可以获取到outputContext。
```
#### 4.设置AI语义解析的语言
```Java
VUIApi.getInstance().setCloudRecognizeLang(String lang);
//设置在线ASR语音识别的语言（见语言对应列表附录）
```
#### 5.设置AI语义解析地理位置上报接口
```Java
 /**
     * List<ScanResult> scanResultList = 
     * 									getApplication()
     * 									.getApplicationContext()
     * 									.getSystemService(Context.WIFI_SERVICE).getScanResults();
     */
     
VUIApi.getInstance().reportLocationInfo(List<ScanResult> scanResultList);

/**
     * 上报地理位置信息接口，用于AI语义的结果使用。比如：询问当前位置的天气、时间等
     *
     * @param latitude  纬度
     * @param longitude 经度
     * @param country
     * @param province
     * @param city
     * @param detail    详细地理位置描述
     *                  例如：
     *                  "latitude": 22.5375738,
     *                  "longitude": 113.9568349,
     *                  "address": {
     *                  "country": "中国",
     *                  "province": "广东省",
     *                  "city": "深圳市",
     *                  "detail": "广东省 深圳市 南山区 科技南十二路 靠近交通银行(深圳高新园支行)"
     * @see ScanResult
     */
     
VUIApi.getInstance().reportLocationInfo(String latitude, String longitude, String country, String province, String city, String detail);
```
## 四.语音播报   
#### 1.开始使用TTS
```Java
VUIApi.getInstance().speak(String text);//text为需要进行语音播报的文本

VUIApi.getInstance().speak(String text, RTTSListener listener);//RTTSListener为TTS监听器

```

#### 2.停止使用TTS
```Java
VUIApi.getInstance().stopSpeak();
```

#### 3.TTS监听器
```Java
new RTTSListener() {
            @Override
            public void onSpeakBegin() {
            //TTS开始
            }

            @Override
            public void onCompleted() {
            //TTS结束
            }

            @Override
            public void onError(int code) {
            }
        }
```
#### 4.设置Speaker
```Java
VUIApi.getInstance().setSpeaker(String speaker);

注意：在线模式设置的是Language,离线模式设置的是发音人的名字．(初始化之后可以动态设置)
```
#### 5.获取TTS音频数据  
```Java
            VUIApi.getInstance().getTTSAudioData(text, new RTTSAudioDataListener() {
                @Override
                public void onSpeakAudio(VUIApi.VUITTSAudioType type, byte[] data, boolean isFinish) {
                    if (type == VUIApi.VUITTSAudioType.TYPE_AUDIO_DATA) {
                      //离线TTS返回data是音频数据
                      //当 isFinish 为 true 时，表示TTS结束,最后一块数据 data 是 null
                    } else {
                      //data是在线的TTS返回的url地址;
                    }
                }
            });  
```

## 五.全局设置
   
#### 1.资源释放
```Java
VUIApi.getInstance().release();
```
#### 2.设置Log等级
```Java
VUIApi.getInstance().setLogLevel(int logLevel);
//logLevel 日志级别0-5,默认是0, 最高级别是5
```








