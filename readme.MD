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

# 이미지분석

먼저 메인 액티비티부터 살펴보겠습니다.

```
motionDetector = new MotionDetector(this, (SurfaceView) findViewById(R.id.surfaceView));
        motionDetector.setMotionDetectorCallback(new MotionDetectorCallback() {
            @Override
            public void onMotionDetected() {
                if(isMotionPossible) {
                    Vibrator v = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
                    v.vibrate(80);
                    //아래로 드래그하기
                    long time = SystemClock.uptimeMillis();
                    int interval = 300;
                    if(isMotionUnder) {
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_DOWN,
                                300,
                                1000,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_MOVE,
                                300,
                                1000 - interval,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_UP,
                                300,
                                1000 - interval,
                                0));
                    }else {
                        //위로 드래그하기
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_DOWN,
                                300,
                                300,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_MOVE,
                                300,
                                300 + interval,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_UP,
                                300,
                                300 + interval,
                                0));
                    }
                    txtStatus.setText("Motion detected");

                }
            }

            @Override
            public void onTooDark() {
                txtStatus.setText("Too dark here");
            }
        });
```
MotionDetector객체를 만듭니다.
해당 객체에서 사용자의 카메라 이미지를 통해 모션을 파악합니다.
motionDetector에 콜백함수가 포함된 객체를 지정해줍니다.
motionDetector를 통해 인식된 결과가 콜백함수에 나옵니다.
따라서 모션이 인식되었다면 적절한 이벤트를 발생시킵니다.
반면 너무 어두우면 모션이 인식되지 않음을 알립니다.

다음은 MotionDetector.java를 살펴보겠습니다.

```
class MotionDetectorThread extends Thread {
        private AtomicBoolean isRunning = new AtomicBoolean(true);

        public void stopDetection() {
            isRunning.set(false);
        }

        @Override
        public void run() {
            while (isRunning.get()) {
                long now = System.currentTimeMillis();
                if (now-lastCheck > checkInterval) {
                    lastCheck = now;

                    if (nextData.get() != null) {
                        int[] img = ImageProcessing.decodeYUV420SPtoLuma(nextData.get(), nextWidth.get(), nextHeight.get());

                        // check if it is too dark
                        int lumaSum = 0;
                        for (int i : img) {
                            lumaSum += i;
                        }
                        if (lumaSum < minLuma) {
                            if (motionDetectorCallback != null) {
                                mHandler.post(new Runnable() {
                                    @Override
                                    public void run() {
                                        motionDetectorCallback.onTooDark();
                                    }
                                });
                            }
                        } else if (detector.detect(img, nextWidth.get(), nextHeight.get())) {
                            // check
                            if (motionDetectorCallback != null) {
                                mHandler.post(new Runnable() {
                                    @Override
                                    public void run() {
                                        motionDetectorCallback.onMotionDetected();
                                    }
                                });
                            }
                        }
                    }
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```
img가 너무 어두우면 onTooDark()이벤트를 보냅니다.
아니면 detect(img, ...)함수에 img를 전달합니다.
그리고 만약 true가 리턴된다면 메인 엑티비티에서 전달해준 콜백객체 내에 있는 onMotionDetected()함수를 실행합니다.
onMotionDetected()함수는 메인 액티비티 파일에 같이 정의해놓았으며 위 혹은 아래 슬라이드 이벤트를 발생시킵니다.
이때 이런 이벤트 요소들은 mHandler를 통해 메인 스레드에서 하나씩 실행됩니다.
그리고 빠른 시간안에 위의 연산들을 다시 수행하지않도록 Sleep(10)을 통해 일정시간 간격을 둡니다.

다음은 AggregateLumaMotionDetection.java를 살펴보겠습니다.

```
 /**
     * Detect motion using aggregate luma values. {@inheritDoc}
     */
    @Override
    public boolean detect(int[] luma, int width, int height) {
        if (luma == null) throw new NullPointerException();

        int[] original = luma.clone();

        // Create the "mPrevious" picture, the one that will be used to check
        // the next frame against.
        if (mPrevious == null) {
            mPrevious = original;
            mPreviousWidth = width;
            mPreviousHeight = height;
            // Log.i(TAG, "Creating background image");
        }

        // long bDetection = System.currentTimeMillis();
        boolean motionDetected = isDifferent(luma, width, height);
        // long aDetection = System.currentTimeMillis();
        // Log.d(TAG, "Detection "+(aDetection-bDetection));

        // Replace the current image with the previous.
        mPrevious = original;
        mPreviousWidth = width;
        mPreviousHeight = height;

        return motionDetected;
    }
```
detect함수는 위에서 제시된 것처럼 전달받은 luma값에 대해서 
이전 이미지의 luma값과 비교해서 모션이 발견되었는지 판단합니다.
이때 isDifferent(...)함수를 사용합니다.
이외에 현재 이미지를 mPrevious 즉 다음에 비교될 이전 이미지 변수에 넣어줍니다.

```
protected boolean isDifferent(int[] first, int width, int height) {
        if (first == null) throw new NullPointerException();

        if (mPrevious == null) return false;
        if (first.length != mPrevious.length) return true;
        if (mPreviousWidth != width || mPreviousHeight != height) return true;

        if (mPreviousState == null) {
            mPreviousState = new State(mPrevious, mPreviousWidth, mPreviousHeight);
            return false;
        }

        State state = new State(first, width, height);
        Comparer comparer = new Comparer(state, mPreviousState, mXBoxes, mYBoxes, mLeniency, mDebugMode);

        boolean different = comparer.isDifferent();
        // String output = "isDifferent="+different;

        mPreviousState = state;

        return different;
    }
```

Comparer생성자에 이전 이미지의 정보, box사이즈, 현재이미지 정보등을 넘겨주고
comparer.isDifferent()함수를 통해 이전 이미지와 큰 차이가 나는지 판단합니다.
이때 판단이 되는 기준값은 mLeniency를 통해 전달됩니다.
그리고 현재 이미지는 다음을 위해 mPreviousState에 저장됩니다.

다음은 Comparer.java를 살펴보겠습니다.

```
public Comparer(State s1, State s2, int xBoxes, int yBoxes, int leniency, int debug) {
        this.state1 = s1;
        this.state2 = s2;

        this.xBoxes = xBoxes;
        if (this.xBoxes > s1.getWidth()) this.xBoxes = s1.getWidth();

        this.yBoxes = yBoxes;
        if (this.yBoxes > s1.getHeight()) this.yBoxes = s1.getHeight();

        this.leniency = leniency;
        this.debugMode = 0;

        // how many points per box
        this.xPixelsPerBox = (int) (Math.floor(s1.getWidth() / this.xBoxes));
        if (xPixelsPerBox <= 0) xPixelsPerBox = 1;
        this.yPixelsPerBox = (int) (Math.floor(s1.getHeight() / this.yBoxes));
        if (yPixelsPerBox <= 0) yPixelsPerBox = 1;

        this.xLeftOver = s1.getWidth() - (this.xBoxes * this.xPixelsPerBox);
        if (xLeftOver > 0) this.xBoxes++;
        yLeftOver = s1.getHeight() - (this.yBoxes * this.yPixelsPerBox);
        if (yLeftOver > 0) this.yBoxes++;

        this.rowWidthInPix = (this.xBoxes * xPixelsPerBox) - (xPixelsPerBox - xLeftOver);
        this.colWidthInPix = this.xPixelsPerBox;

        this.debugMode = debug;

        this.different = isDifferent(this.state1, this.state2);
    }
```
다음과 같이 이전 이미지, 현재 이미지, 기준 box갯수, leniency가 넘어오는데
각각의 박스마다 이전 이미지와 현재 이미지를 비교할 것인데
박스당 몇 픽셀이 들어갈 것인지 연산합니다.

```
public boolean isDifferent(State s1, State s2) {
        if (s1 == null || s2 == null) throw new NullPointerException();
        if (s1.getWidth() != s2.getWidth() || s1.getHeight() != s2.getHeight()) return true;

        // Boxes
        this.variance = new int[yBoxes][xBoxes];

        // set to a different by default, if a change is found then flag
        // non-match
        boolean different = false;
        // loop through whole image and compare individual blocks of images
        int b1 = 0;
        int b2 = 0;
        int diff = 0;
        for (int y = 0; y < yBoxes; y++) {
            for (int x = 0; x < xBoxes; x++) {
                b1 = aggregateMapArea(state1.getMap(), x, y);
                b2 = aggregateMapArea(state2.getMap(), x, y);
                diff = Math.abs(b1 - b2);
                variance[y][x] = diff;
                // the difference in a certain region has passed the threshold
                // value
                if (diff > leniency) different = true;
            }
        }
        return different;
    }
```
총 확인할 박스 갯수만큼 이전 이미지 박스 내 luma값, 현재 이미지 박스 내 luma값들의
평균을 diff에 저장하고 만약 임계값을 넘으면 모션이 발견됬다고 인지합니다.

```
private int aggregateMapArea(int[] map, int xBox, int yBox) {
        if (map == null) throw new NullPointerException();

        int yPix = yPixelsPerBox;
        int xPix = xPixelsPerBox;
        if (yBox == (yBoxes - 1) && yLeftOver > 0) yPix = yLeftOver;
        if (xBox == (xBoxes - 1) && xLeftOver > 0) xPix = xLeftOver;

        int rowOffset = (yBox * yPixelsPerBox * rowWidthInPix);
        int columnOffset = (xBox * colWidthInPix);

        int i = 0;
        int iy = 0;
        for (int y = 0; y < yPix; y++) {
            iy = (y * (xBoxes * xPixelsPerBox)) - (y * (xPixelsPerBox - xLeftOver));
            for (int x = 0; x < xPix; x++) {
                i += map[rowOffset + columnOffset + iy + x];
            }
        }

        return (i / (xPix * yPix));
    }
```
지정한 박스좌표를 주고 해당 박스내의 모든 luma값을 더해 
평균을 내어 리턴합니다.


# 터치 이벤트

터치이벤트는 webview내에서만 발생시킵니다.

```
 motionDetector = new MotionDetector(this, (SurfaceView) findViewById(R.id.surfaceView));
        motionDetector.setMotionDetectorCallback(new MotionDetectorCallback() {
            @Override
            public void onMotionDetected() {
                if(isMotionPossible) {
                    Vibrator v = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
                    v.vibrate(80);
                    //아래로 드래그하기
                    long time = SystemClock.uptimeMillis();
                    int interval = 300;
                    if(isMotionUnder) {
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_DOWN,
                                300,
                                1000,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_MOVE,
                                300,
                                1000 - interval,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_UP,
                                300,
                                1000 - interval,
                                0));
                    }else {
                        //위로 드래그하기
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_DOWN,
                                300,
                                300,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_MOVE,
                                300,
                                300 + interval,
                                0));
                        webView.dispatchTouchEvent(MotionEvent.obtain(time,
                                time,
                                MotionEvent.ACTION_UP,
                                300,
                                300 + interval,
                                0));
                    }
                    txtStatus.setText("Motion detected");

                }
            }

            @Override
            public void onTooDark() {
                txtStatus.setText("Too dark here");
            }
        });
```
모션이 인식되면 지정한 방향(위 혹은 아래)로 슬라이드합니다.
ACTION_DOWN으로 터치를 시작하고
터치된 상태에서 ACTION_MOVE를 통해 이동하고
ACTION_UP을 통해 터치를 뗍니다.
그러면 ACTION_MOVE의 방향에 따라 위 혹은 아래로 슬라이드가 됩니다.

다음은 make_Touche(...)메소드에 대해서 알아보겠습니다.
```
if(msg.contains("터치")) {
            String[] array = msg.split(" ");
            if (array.length != 3) return;
            int x,y;
            x=y=0;
            if(msg.contains("백")) {
                x = 100; y=100;
            }
            else if(!CheckNumber(array[1]) || !CheckNumber(array[2])){return;}

            if(x!=100 && y!=100) {
                x = Integer.parseInt(array[1]);
                y = Integer.parseInt(array[2]);
            }
            if (x < 0 || x > 1000 || y < 0 || y > 1000) return;
            //Toast.makeText(getApplicationContext(), Integer.toString(x) + " " + Integer.toString(y) + "\n", Toast.LENGTH_SHORT).show();
            webView.dispatchTouchEvent(MotionEvent.obtain(SystemClock.uptimeMillis(),
                    SystemClock.uptimeMillis(),
                    MotionEvent.ACTION_DOWN,
                    x,
                    y,
                    0));
            webView.dispatchTouchEvent(MotionEvent.obtain(SystemClock.uptimeMillis(),
                    SystemClock.uptimeMillis(),
                    MotionEvent.ACTION_UP,
                    x,
                    y,
                    0));
        }
```
명령은 "터치 좌표x 좌표y"로 되어있습니다.
따라서 적절하게 x,y좌표를 파싱해주어 숫자로 변환합니다.
그리고 해당 좌표에 터치이벤트를 발생시킵니다.

```
else if(msg.contains("아래로 드래그")) {
            String[] array = msg.split(" ");
            Toast.makeText(getApplicationContext(),"" + SystemClock.currentThreadTimeMillis(),Toast.LENGTH_SHORT).show();
            time = SystemClock.uptimeMillis();
            int interval;
            if (array.length == 3) {
                if (!CheckNumber(array[2])) return;
                interval = Integer.parseInt(array[2]);
                if (interval < 0 || interval > 1000) return;
            }else if(array.length==2){
                interval=300;
            }else return;
            webView.dispatchTouchEvent(MotionEvent.obtain(time,
                    time,
                    MotionEvent.ACTION_DOWN,
                    300,
                    1000,
                    0));
            webView.dispatchTouchEvent(MotionEvent.obtain(time,
                    time,
                    MotionEvent.ACTION_MOVE,
                    300,
                    1000-interval,
                    0));
            webView.dispatchTouchEvent(MotionEvent.obtain(time,
                    time,
                    MotionEvent.ACTION_UP,
                    300,
                    1000-interval,
                    0));

        }
```
아래로 위로 드래그는 위에서 설명드린 슬라이드과 같습니다.

```
else if(msg.contains("매트릭스")){

            String[] array = msg.split(" ");
            if (array.length != 2) return;
            int interval = Integer.parseInt(array[1]);

            if(interval==100) {
                if (FL_matrix100.getVisibility() == View.VISIBLE) {
                    FL_matrix100.setVisibility(View.INVISIBLE);
                } else {
                    FL_matrix100.setVisibility(View.VISIBLE);
                }
            }
            else if(interval==50){
                if (FL_matrix50.getVisibility() == View.VISIBLE) {
                    FL_matrix50.setVisibility(View.INVISIBLE);
                } else {
                    FL_matrix50.setVisibility(View.VISIBLE);
                }
            }
//            if(msg.contains("보여")) {
//                FL_matrix.setVisibility(View.VISIBLE);
//            }else if(msg.contains("없애")){
//                FL_matrix.setVisibility(View.INVISIBLE);
//            }
        }
```
사용자가 터치 이벤트시 적절한 좌표를 파악하는데 문제가 있기때문에
100단위, 50단위 메트릭스를 띄울지 말지 판단하는 부분입니다.

```
else if(msg.contains("키보드") || msg.contains("keyboard")){
            //onSearchRequested();
            String[] array = msg.split(" ");
            if(array.length!=2) return;
            String srchWord = array[0];
            for(int i=0;i<srchWord.length();i++){
                makeKeyEvent(srchWord.charAt(i));
            }
            //webView.dispatchKeyEvent(new KeyEvent(SystemClock.uptimeMillis()+500,SystemClock.uptimeMillis() + 1000,KeyEvent.ACTION_UP,KeyEvent.KEYCODE_ASSIST,0));
        }
```
키보드를 음성으로 입력하는 부분인데 명확히 작동하지는 않습니다.

터치이벤트외에 다른 이벤트도 적게 되었는데 다른 것도 동일합니다.



        