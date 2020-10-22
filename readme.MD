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
RecognitionListener는 SpeechRecognizer의 인식 관련 이벤트를 관리합니다.<br>
RecognizerIntent는 SpeechRecognizer를 실행시키위해서 존재합니다.<br>
SpeechRecognizer는 음성인식 및 텍스트변환을 위해 사용합니다.<br>

```
private SpeechRecognizer speech = null;
```
SpeechRecognizer변수를 선언합니다.<br>

```
private Intent recognizerIntent;
```
RecognizerIntent를 담기위한 Intent 변수를 만듭니다.<br>

```
requestAudioPermissions();
```
오디오 권한을 얻습니다.<br>

```
speech = SpeechRecognizer.createSpeechRecognizer(this);
speech.setRecognitionListener(this);
```
현재 Activity(this)에 대해서 SpeechRecognizer를 만들고<br>
RecognitionListener를 implements한 현재 Activity(this)를<br>
speech(SpeechRecognizer)의 RecognitionListener로 설정한다.<br>

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

SpeechRecognizer를 실행시키기 위한 설정 정보를 담기위해 <br>
RecognizerIntent를 변수에 저장하고<br>
언어 등 기타 설정을 넣어준다.<br>

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
토글버튼 체크 여부에 따라 speech(SpeechRecognizer) 실행 여부 결정<br>
이때 체크되어 있다면 RecognizerIntent의 전달과 함께 실행<br>

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
3초마다 toggle을 다시 체크해준다.<br>
계속 지속적으로 사용자의 음성을 듣기위해서<br>

```
@Override
    public void onBeginningOfSpeech() {
        Log.i(LOG_TAG, "onBeginningOfSpeech");
        progressBar.setIndeterminate(false);
        progressBar.setMax(10);
   	}
```
현재 Activity는 RecognizerListener가 implements되어 있기 때문에<br>
onBeginningOfSpeech메소드를 통해 상태바를 작동시켜 <br>
사용자에게 현재 음성을 듣고 있다는 것을 알려줍니다.<br>

```
@Override
    public void onEndOfSpeech() {
        Log.i(LOG_TAG, "onEndOfSpeech");
        progressBar.setIndeterminate(true);
        toggleButton.setChecked(false);
    }
```
듣기를 마치면 상태바를 없애고<br>
토글버튼 체크를 해제합니다.<br>
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
음성듣기를 마치면<br>
인식된 단어들을 matches에 넣어주고<br>
matches의 첫번째 String에는 SpeechRecognizer의 결과들이 들어있습니다.<br>
따라서 toast메시지에 해당 값을 띄워주고<br>
또한 해당 String에 따라 올바른 터치명령 및 모션 기능 활성화를 판단하기위해 <br>
make_Touch메소드에 String을 넘겨줍니다.<br>

```
public static String getErrorText(int errorCode) {
        String message;
        switch (errorCode) {
            case SpeechRecognizer.ERROR_AUDIO:
                message = "Audio recording error";
                break;
            case SpeechRecognizer.ERROR_CLIENT:
                message = "Client side error";
                break;
            case SpeechRecognizer.ERROR_INSUFFICIENT_PERMISSIONS:
                message = "Insufficient permissions";
                break;
            case SpeechRecognizer.ERROR_NETWORK:
                message = "Network error";
                break;
            case SpeechRecognizer.ERROR_NETWORK_TIMEOUT:
                message = "Network timeout";
                break;
            case SpeechRecognizer.ERROR_NO_MATCH:
                message = "No match";
                break;
            case SpeechRecognizer.ERROR_RECOGNIZER_BUSY:
                message = "RecognitionService busy";
                break;
            case SpeechRecognizer.ERROR_SERVER:
                message = "error from server";
                break;
            case SpeechRecognizer.ERROR_SPEECH_TIMEOUT:
                message = "No speech input";
                break;
            default:
                message = "Didn't understand, please try again.";
                break;
        }
        return message;
    }
```
SpeechRecognizer의 적절한 에러에 해당하는 메시지를 리턴<br>

```
private final int MY_PERMISSIONS_RECORD_AUDIO = 1;

    private void requestAudioPermissions() {
        if (ContextCompat.checkSelfPermission(this,
                Manifest.permission.RECORD_AUDIO)
                != PackageManager.PERMISSION_GRANTED) {

            //When permission is not granted by user, show them message why this permission is needed.
            if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                    Manifest.permission.RECORD_AUDIO)) {
                Toast.makeText(this, "Please grant permissions to record audio", Toast.LENGTH_LONG).show();

                //Give user option to still opt-in the permissions
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.RECORD_AUDIO},
                        MY_PERMISSIONS_RECORD_AUDIO);

            } else {
                // Show user dialog to grant permission to record audio
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.RECORD_AUDIO},
                        MY_PERMISSIONS_RECORD_AUDIO);
            }
        }
        //If permission is granted, then go ahead recording audio
        else if (ContextCompat.checkSelfPermission(this,
                Manifest.permission.RECORD_AUDIO)
                == PackageManager.PERMISSION_GRANTED) {

            //Go ahead with recording audio now
        }
    }
```
위에서 사용된 requestAudioPermission()메소드 구현부분입니다.<br>
context를 통해 오디오 녹음 권한을 얻었는지 확인합니다.<br>
얻기 못했다면 사용자에게 현재 액티비티의 오디오 녹화 권한을 요청합니다.<br>

```
@Override
    protected void onPause() {
        motionDetector.onPause();
        super.onPause();
        if (speech != null) {
            speech.destroy();
            Log.i(LOG_TAG, "destroy");
        }
    }
```
앱이 멈추게 된다면<br>
speech를 없앱니다. <br>

        