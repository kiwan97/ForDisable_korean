## For Disabled, 장애인을 위한 안드로이드 웹 브라우져
몸이 불편하신 장애인분들을 위한 음성, 모션을 활용한 안드로이드 웹 브라우져 입니다.

Motion detection from : https://github.com/jjoe64/android-motion-detection

## 사용된 기술
# 음성인식
android.speech 라이브러리 내부의 RecognitionListener, RecognizerIntent, SpeechRecognizer를 사용<br>
# 이미지분석
Motion detection from : https://github.com/jjoe64/android-motion-detection<br>
# 터치 이벤트
android.webview의 dispatchTouchEvent(...) 사용<br>

## 세부설명
# 음성인식
```
import android.speech.RecognitionListener;
import android.speech.RecognizerIntent;
import android.speech.SpeechRecognizer;
```
RecognitionListener는 SpeechRecognizer의 인식 관련 이벤트를 관리합니다.
RecognizerIntent는 SpeechRecognizer를 실행시키위해서 존재합니다.
SpeechRecognizer는 음성인식 및 텍스트변환을 위해 사용합니다.

```
private SpeechRecognizer speech = null;
```
SpeechRecognizer변수를 선언합니다.

```
private Intent recognizerIntent;
```
RecognizerIntent를 담기위한 Intent 변수를 만듭니다.

```
requestAudioPermissions();
```
오디오 권한을 얻습니다.

```
speech = SpeechRecognizer.createSpeechRecognizer(this);
speech.setRecognitionListener(this);
```
현재 Activity(this)에 대해서 SpeechRecognizer를 만들고
RecognitionListener를 implements한 현재 Activity(this)를
speech(SpeechRecognizer)의 RecognitionListener로 설정한다.

```
recognizerIntent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
recognizerIntent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_PREFERENCE,"ko-KR");
recognizerIntent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_PREFERENCE,"en-US");
recognizerIntent.putExtra("android.speech.extra.EXTRA_ADDITIONAL_LANGUAGES", new String[]{"en-US"});
recognizerIntent.putExtra(RecognizerIntent.EXTRA_CALLING_PACKAGE,this.getPackageName());
recognizerIntent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,RecognizerIntent.LANGUAGE_MODEL_WEB_SEARCH);
recognizerIntent.putExtra(RecognizerIntent.EXTRA_CALLING_PACKAGE,this.getPackageName());

recognizerIntent.putExtra(RecognizerIntent.EXTRA_MAX_RESULTS, 3); 
```

SpeechRecognizer를 실행시키기 위한 설정 정보를 담기위해 
RecognizerIntent를 변수에 저장하고
언어 등 기타 설정을 넣어준다.

```
toggleButton.setOnCheckedChangeListener(new OnCheckedChangeListener() {

    @Override
    public void onCheckedChanged(CompoundButton buttonView,
                                 boolean isChecked) {
        if (isChecked) {
            progressBar.setVisibility(View.VISIBLE);
            progressBar.setIndeterminate(true);
            speech.startListening(recognizerIntent);
            make_Touch("");
        } else {
            progressBar.setIndeterminate(false);
            progressBar.setVisibility(View.INVISIBLE);
            speech.stopListening();
        }
    }
});
```
토글버튼 체크 여부에 따라 speech(SpeechRecognizer) 실행 여부 결정
이때 체크되어 있다면 RecognizerIntent의 전달과 함께 실행

```
private class TimerHandler extends Handler{
    @Override
    public void handleMessage(Message msg){
        switch (msg.what) {
            case MESSAGE_TIMER_START:
                if(!toggleButton.isChecked())
                    toggleButton.setChecked(true);

                this.sendEmptyMessageDelayed(MESSAGE_TIMER_START,3000);
                break;

        }

    }
}
```
3초마다 toggle을 다시 체크해준다.
계속 지속적으로 사용자의 음성을 듣기위해서

```
@Override
    public void onBeginningOfSpeech() {
        Log.i(LOG_TAG, "onBeginningOfSpeech");
        progressBar.setIndeterminate(false);
        progressBar.setMax(10);
   	}
```
현재 Activity는 RecognizerListener가 implements되어 있기 때문에
onBeginningOfSpeech메소드를 통해 상태바를 작동시켜 
사용자에게 현재 음성을 듣고 있다는 것을 알려줍니다.

```
@Override
    public void onEndOfSpeech() {
        Log.i(LOG_TAG, "onEndOfSpeech");
        progressBar.setIndeterminate(true);
        toggleButton.setChecked(false);
    }
```
듣기를 마치면 상태바를 없애고
토글버튼 체크를 해제합니다.
```
@Override
    public void onResults(Bundle results) {
        Log.i(LOG_TAG, "onResults");
        ArrayList<String> matches = results.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION);
//        String text = "";
//        for (String result : matches)
//            text += result + "\n";
        text.setText(matches.get(0));
        toast.show();
        //returnedText.setText(matches.get(0));
        make_Touch(matches.get(0));
    }
```
음성듣기를 마치면
인식된 단어들을 matches에 넣어주고
        