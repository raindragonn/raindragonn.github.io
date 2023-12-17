---
layout: "post"
title: "[Android] Text To Sppech(TTS)"
subtitle: "Text To Sppech(TTS)"
date: 2023-12-17
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
  - Android
  - TTS
---

## [Android] Text To Speech (TTS)

## TTS란?

Text To Speech의 약자로 텍스트를 음성으로 출력해주는 것을 말한다.

본 예제는 안드로이드에서 기본적으로 지원해주는 `Android.speech.tts` 를 사용한다.

## 사용법

사용 방법은 초기화, 설정, 출력으로  매우 간단하다.

### 1. 초기화

`TextToSpeech(Context context, OnInitListener listener)` 를 이용해 초기화 한다.

두번 째 매개변수 `listener` 는 해당 TTS의 초기화 완료 콜백을 위한 인터페이스이다. 해당 인터페이스의 `OnInit`을 통해 초기화 성공, 실패 여부를 콜백으로 알 수 있다.

### 2. 음성 설정

음성 설정은 부가적인 설정으로, TTS의 음성 설정 등이 가능하다.

- 음성 언어 설정 `TextToSpeech.setLanguage(Locale)`
- 음성 톤 설정 `TextToSpeech.setPitch(Float)`
    - 1.0f 가 기본 설정이며, 낮출 수록 톤이 낮아진다
    - 직접 확인한 바에 따르면 큰 차이는 느끼지 못하겠다.
- 음성 속도 설정 `TextToSpeech.setSpeechRate(float)`
    - 1.0f 가 기본 설정이며 높일수록 음성이 빨라진다.

### 3. 음성 출력

`TextToSpeech.speak(CharSequence text, int queueMode,Bundle params,String utteranceId)` 을 통해 음성을 출력할 수 있다.

**[ 매개변수 설명 ]** 

- text: 음성으로 출력 하고자 하는 텍스트
- queueMode:  재생 대기열의 처리 방법 ( 쉽게 말해 음성 이 출력되는 진행중에 음성 출력을 하는 경우 음성에 대한 처리 방법)
    - *`QUEUE_FLUSH` :  현재 진행중이던 음성 및 대기열을 없애고 새로운 음성을 출력한다.*
    - *`QUEUE_ADD` : 새로운 음성을 대기열에 추가하고 음성이 끝나면 이후 출력한다.*
- params: Volume, Stream 등의 변경을 요청할 수 있으며, 요청할 것이 없는경우 null로 사용한다.
- utteranceId: 유니크한 아이디값

## 전체 코드

```kotlin
class TtsManagerImpl(
    private val _context: Context
) : UtteranceProgressListener(), TtsManager,
    TextToSpeech.OnInitListener {

    private var _tts: TextToSpeech? = null
    private var _onSpeakDoneListener: (() -> Unit)? = null
    private var _lastSpeakText: String? = null

    private val tts
        get() = _tts ?: TextToSpeech(_context, this)
            .also { _tts = it }

    override fun onStart(utteranceId: String?) {
        _lastSpeakText = null
    }

    override fun onDone(utteranceId: String?) {
        _onSpeakDoneListener?.invoke()
        _onSpeakDoneListener = null
        _lastSpeakText = null
    }

    override fun onError(utteranceId: String?) {
        Log.d("DEV_LOG", "MyTtsImpl.kt.onError: ")
    }

    private var id = 0
    override fun onInit(status: Int) {
        if (status == TextToSpeech.SUCCESS) {
            _tts?.language = Locale.getDefault()
            _lastSpeakText?.let { speakText ->
                tts.speak(speakText, TextToSpeech.QUEUE_FLUSH, null, "${++id}")
            }
        } else {
            release()
        }
    }

    override fun setPitch(pitch: Float) {
        tts.setPitch(pitch)
    }

    override fun speak(speakText: String, doneListener: (() -> Unit)?) {
        doneListener?.let {
            _onSpeakDoneListener = it
            tts.setOnUtteranceProgressListener(this)
        }
        tts.speak(speakText, TextToSpeech.QUEUE_FLUSH, null, "${++id}").let { result ->
            if (result == TextToSpeech.ERROR) _lastSpeakText = speakText
        }
    }

    override fun release() {
        _tts?.apply {
            stop()
            shutdown()
        }
        _onSpeakDoneListener = null
        _tts = null
    }
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    private val _ttsManager: TtsManager by lazy { TtsManagerImpl(baseContext) }

    private lateinit var _btnSpeech: Button
    private lateinit var _etText: EditText
    private lateinit var _tvPitch: TextView
    private lateinit var _sbPitch: SeekBar

    private val _text
        get() = _etText.text.toString()

    companion object {
        const val MAX_PITCH = 1
        const val PITCH_STEP = 0.1f
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initViews()
    }

    override fun onDestroy() {
        _ttsManager.release()
        super.onDestroy()
    }

    private fun initViews() {
        _etText = findViewById(R.id.et_text)
        _tvPitch = findViewById(R.id.tv_pitch)
        _sbPitch = findViewById(R.id.sb_pitch)
        _btnSpeech = findViewById(R.id.btn_speech)
        _btnSpeech.setOnClickListener(::onClickBtn)

        _sbPitch.max = (MAX_PITCH / PITCH_STEP).toInt()
        _sbPitch.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
            override fun onProgressChanged(seekBar: SeekBar?, progress: Int, fromUser: Boolean) {
                seekBar ?: return
                val pitch = progress * PITCH_STEP
                val format = "%.1f".format(pitch)
                _tvPitch.text = "피치 조절 : $format"
                _ttsManager.setPitch(pitch)
            }

            override fun onStartTrackingTouch(seekBar: SeekBar?) {}
            override fun onStopTrackingTouch(seekBar: SeekBar?) {}
        })
    }

    private fun onClickBtn(view: View) {
        _ttsManager.speak(_text) {
            Log.d("DEV_LOG", "done")
        }
    }
}
```

### [전체 코드 확인](https://github.com/raindragonn/TtsStudy)

### 참고 자료

---

https://developer.android.com/reference/android/speech/tts/TextToSpeech