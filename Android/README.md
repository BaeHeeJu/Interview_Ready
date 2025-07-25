# 🤖 Android 가이드 : 개념 및 개발

Android 개발에 필요한 핵심 개념, 구조, 실무 적용 예시들을 정리한 문서입니다.

---

## 📘 주요 개념

### Activity 생명주기
- 정의
  + Activity 는 사용자와 화면 단위로 상호작용하는 컴포넌트
  + 사용자가 앱 실행/나가기/회전 등 에 따라 Activity는 여러 상태를 거치며 전환
  + 액티비티 생명주기에 대한 이해를 통해 리소스 누수나 사용자 경험 저하 방지 필요
- 상세설명
  + | 메서드             | 호출 시점                          | 주로 하는 일                       |
    | --------------- | ------------------------------ | ----------------------------- |
    | **onCreate()**  | Activity 최초 생성 시               | 뷰 바인딩, 초기 변수 설정, 리소스 할당       |
    | **onStart()**   | Activity가 화면에 보이기 시작할 때        | UI 준비 (아직 포커스는 없음)            |
    | **onResume()**  | Activity가 포커스를 얻고 사용자와 상호작용 가능 | 애니메이션 시작, 센서 리스너 등록           |
    | **onPause()**   | 다른 Activity가 앞에 나올 때 (부분 가림)   | 리소스 해제 (예: 센서, 카메라 등)         |
    | **onStop()**    | 완전히 화면에서 보이지 않을 때              | DB 저장, 리소스 정리, Broadcast 해제 등 |
    | **onRestart()** | 정지되었다가 다시 보여질 때                | 필요한 UI 갱신                     |
    | **onDestroy()** | Activity 종료되거나 시스템에 의해 제거될 때   | 모든 리소스 해제, GC 대상화             |
- 생명주기 상태 예시
  +  앱 시작 시 `onCreate()` → `onStart()` → `onResume()`
  +  다른 앱 열어서 가릴 때 `onPause()` → `onStop()`
  +  다시 돌아왔을 때 `onRestart()` → `onStart()` → `onResume()`
  +  앱 완전 종료 또는 시스템에 의한 제거 `onPause()` → `onStop()` → `onDestroy()`
- 핵심 포인트
  + `onCreate()` - UI 설정
  + `onPause()` - 리소스(센서, GPS, 애니메이션) 해제
  + `onStop()` - 저장이 필요한 데이터 처리
  + 화면 회전 시 
    * `onPause()` → `onStop()` → `onSaveInstanceState` → `onDestroy()`
    * →`onCreate()` → `onStart()` → `onRestoreInstanceState()` → `onResume()`
    * 왜 액티비티는 재생성 할까?
      1. 레이아웃을 다시 계산해야하는데 기존을 파괴하고 새로 만드는 것이 깔끔한 방법
      2. 이전 상태 유지시 레이아웃도 꼬이고 메모리 누수 위험이 커짐
- Cold/Warm/Hot Start 
  + | 구분             | 정의                       | Activity 생명주기 흐름                           | 특징                               |
    | -------------- | ------------------------ | ------------------------------------------ | -------------------------------- |
    | **Cold Start** | 앱이 완전히 종료된 상태에서 처음 실행됨   | `onCreate()` → `onStart()` → `onResume()`  | 리소스 로딩, Application 객체 생성, 가장 느림 |
    | **Warm Start** | 앱이 백그라운드에 있다가 다시 돌아옴     | `onRestart()` → `onStart()` → `onResume()` | Activity는 살아있고, 앱 프로세스도 유지됨      |
    | **Hot Start**  | 화면 꺼짐, 잠깐 홈으로 나갔다가 다시 복귀 | `onResume()` 단독                            | UI도 메모리에 있고, 거의 즉시 복구됨           |
  + Cold Start 최적화
    * Application 에서 무거운 초기화 지양
    * SplashActivity 최소화
    * 비동기 초기화 (WorkManager, Coroutine, Executors)
  + Warm Start 대응 : 재개 시 필요한 데이터만 빠르게 로딩
  + Hot Start 대응 : Resume 시 불필요한 연산 배제


---


### Fragment 생명주기
- 정의
  + Fragment는 자체적인 생명주기를 가지는 UI 구성요소로, Activity 내에서 모듈식 UI를 구축하는 데 사용 됩니다.
  + Fragment의 생명주기를 이해하는 것은 리소스를 올바르게 관리하고 사용자 경험을 향상시키는 데 매우 중요합니다.

- onAttach()
  + Fragment가 Activity에 연결될 때 호출됩니다.
  + 이 시점에서 Fragment는 Activity의 Context를 얻을 수 있습니다.
  + 주로 Activity와 통신을 위한 인터페이스를 초기화하거나, Activity로부터 필요한 데이터를 전달받는 작업을 수행합니다.

- onCreate()
  + Fragment가 생설될 때 호출됩니다.
  + Fragment의 핵심 로직을 초기화하는 작업을 수행합니다.

- onCreateView()
  + Fragment의 UI 레이아웃을 생성하고 반환할 때 호출됩니다.
  + XML 레이아웃 파일을 inflate하여 Fragment의 뷰 계층 구조를 만듭니다.
  + 반환된 뷰가 화면에 표시 됩니다.

- onViewCreated()
  + onCreateView()에서 반환된 뷰가 완전히 생성된 후 호출 됩니다.
  + 이 시점부터 Fragment의 뷰에 안전하게 접근하여 초기화 작업을 수행 할 수 있습니다.

- onStart()
  + Fragment가 사용자에게 보이기 시작할 때 호출됩니다.

- onResume()
  + Fragment가 사용자와 상호작용할 수 있는 상태가 될때 호출됩니다.

- onStop()
  + Fragment가 더 이상 사용자에게 보이지 않게 될 때 호출 됩니다.
  + 화면에 보이지 않는 동안 필요하지 않은 리소스를 해제합니다.

- onSaveInstanceState()
  + Fragment의 상태가 예기치 않게 소멸 될 가능성이 있을 때, 현재 상태를 저장하기 위해 호출 됩니다.
  + Bundle 객체에 저장할 데이터를 넣어두면, 나중에 onCreate(), onCreateView(), onViewCreated()에서 해당 Bundle을 통해 상태를 복원할 수 있습니다.

- onDestroyView()
  + Fragment와 관련된 뷰가 제거될 때 호출됩니다.

- onDestroy()
  + Fragment가 완전히 소멸될 때 호출됩니다.
  + Fragment의 모든 리소스를 정리하고, 백그라운드 스레드를 중지하는 등의 최종 정리 작업을 수행합니다.
  + 이후 해당 Fragment의 인스턴스는 더 이상 사용되지 않습니다.

- onDetach()
  + Fragment가 Activity와의 연결이 끊어질 때 호출됩니다.

- Cold Start
  + 앱이 완전히 종료된 상태에서 처음 실행할 때 입니다.
  + `onAttach()` → `onCreate()`→ `onCreateView()`→ `onViewCreated()`→ `onStart()`→ `onResume()`

- Warm Start
  + 앱의 프로세스는 살아있지만, Activity가 메모리에서 제거되었거나,백그라운에 있다가 다시 시작될 때 입니다.
  + `onStart()`→ `onResume()`

- Hot Start
  + 앱의 프로세스와 Activity가 모두 메모리에 살아있고, 단순히 화면의 전면으로 다시 가져오는 경우 입니다.
  + `onResume()`


---


### Content Provider
- 정의
  + Content Provider는 애플리케이션이 자신의 데이터를 다른 애플리케이션과 안전하게 공유할 수 있도록 관리하는 컴포넌트 입니다.
  + 데이터는 파일 시스템, SQLite 데이터베이스, 웹 또는 애플리케이션이 접근할 수 있는 다른 영구 저장 위치에 저장될 수 있습니다.

- 데이터 공유
  + 애플리케이션 간에 구조화된 데이터를 공유하기 위한 표준 인터페이스를 제공합니다.
  + 안드로이드 시스템의 주소록, 미디어 스토어 등은 Content Provider를 통해 다른 앱에 데이터를 제공합니다.

- 데이터 캡슐화
  + 데이터의 실제 저장 방식을 숨기고, 추상화된 인터페이스를 통해 데이터에 접근 하도록 합니다.
  + 데이터 저장 방식이 변경되더라도 다른 앱의 코드에 영향을 주지 않고 데이터를 제공할 수 있습니다.

- 데이터 접근제어
  + 읽기/쓰기 권한을 세밀하게 제어하여 다른 애플리케이션이 데이터에 접근하는 것을 허용하거나 제한할 수 있습니다.

- 표준 CRUD 인터페이스
  + 데이터를 조작하기 위한 표준 메서드 집합을 제공합니다.
    * query() : 데이터 검색
    * insert() : 새 데이터 삽입
    * update() : 기존 데이터 수정
    * delete() : 데이터 삭제
    * getType() : 특정 URI에 해당하는 데이터의 MIME 타입을 반환

- 콘텐트 URI
  + Content Provider 내의 특정 데이터를 식별하기 위한 고유한 URI 체계를 사용합니다. 
  + content:// : 스킴, 콘텐츠 프로바이더임을 나타냅니다.
  + <authority>: 콘텐츠 프로바이더를 식별하는 고유한 이름
  + <path>: 프로아비더가 제공하는 데이터의 종류를 나타내는 경로
  + <optional_id>: 특정 레코드를 식별하기 위한 ID

- ContentResolver
  + 다른 애플리케이션은 ContentsResolver 객체를 사용하여 특정 콘텐츠 프로아비더와 상호작용합니다.
  + 클라이언트는 ContentResolver의 메서드를 호출하고 ContentResolver는 해당 URI를 기반으로 적절한 콘텐츠 프로바이더를 찾아 요청을 전달 합니다.

- 주의할 점
  + 콘텐츠 프로바이더의 메서드(특히 query())는 시간이 오래 걸릴 수 있으므로, UI 스레드에서 직접 호출하면 ANR이 발생할 수 있습니다. 비동기적으로 처리해야 합니다.
  + 데이터 공유 시 보안에 유의해야 합니다. 필요한 최소한의 권한만 부여하고, 민감한 정보는 신중하게 다루어야 합니다.
  + 자신의 앱 내에서만 데이터를 사용하는 경우에는 굳이 콘텐츠 프로바이더를 만들 필요가 없을 수도 있습니다.


---


### Service
- 정의
  + 서비스는 사용자와 직접적인 UI 상호작용 없이 백그라운드에서 오래 실행되는 작업을 수행하거나, 
    다른 애플리케이션에 기능을 제공하기 위한 애플리케이션에 기능을 제공하기 위한 애플리케이션 컴포넌트 입니다.

- 주요 특징 및 용도
  + 백그라운드 작업
    * 네트워크 트랜잭션 처리
    * 음악 재생
    * 백그라운드 데이터 동기화
    * 위치 정보 업데이트
  + UI 없음
    * 서비스는 자체적인 사용자 인터페이스를 가지지 않습니다.
  + 생명주기
    * 액티비티와는 다른 독립적인 생명주기를 가집니다.
  + 포그라운드 서비스
    * 사용자에게 현재 실행 중임을 인지시켜야 하는 중요한 작업을 수행하는 서비스입니다.
    * 반드시 상태 표시줄에 알림을 표시해야 합니다.
    * 시스템이 메모리 부족 상황에서도 포그라운드 서비스를 쉽게 종료시키지 않습니다.
    * 안드로이드 9 버전 이상에서는 포그라운드 서비스 실행시 FOREGROUND_SERVICE 권한이 필요합니다.
  + 서비스는 기본적으로 애플리케이션의 메인 스레드에서 실행됩니다. 따라서 서비스 내에서 CPU를 많이 사용하거나 시간이 오래 걸리는 작업을 직접 수행하면 ANR 오류가 발생할 수 있습니다.
  + 백그라운드 실행 제한 : 최신안드로이드 버전에서는 배터리 수명과 시스템 성능을 위해 백그라운드 서비스 실행에 제한이 강화되었습니다. 따라서 장기 실행 작업에서는 WorkManager와 같은 다른 솔루션을 고려하는 것이 좋습니다.


---


## 4대 컴포넌트 Activity
- 정의
  + 안드로이드에서 하나의 사용자 인터페이스 화면
  + 사용자와 직접 상호작용하는 컴포넌트이며 UI를 구성하고 앱 흐름을 제어하는 기본 단위
- 특징
  + | 항목                     | 내용                                       |
    | ---------------------- | ---------------------------------------- |
    | **UI 구성 단위**           | 앱의 화면 하나가 하나의 Activity                   |
    | **생명주기 존재**            | `onCreate()`부터 `onDestroy()`까지 상태 전환     |
    | **명시적/암시적 인텐트로 호출 가능** | 다른 Activity나 앱 호출 가능                     |
    | **Manifest 등록 필수**     | `AndroidManifest.xml`에 선언돼야 실행 가능        |
    | **백스택 관리**             | `startActivity()` → 이전 Activity는 백스택에 저장 |
    | **Context 역할 수행**      | 리소스 접근, 시스템 서비스 호출 가능                    |
- Intent
  + 명시적(Intent Explicit) 특정 컴포넌트 지정
    * ```kotlin
      val intent = Intent(this, DetailActivity::class.java)
      startActivity(intent)
      ```
  + 암시적(Intent Implicit) 작업(Action)에 맞는 컴포넌트 자동 매칭
    * ```kotlin
      val intent = Intent(Intent.ACTION_VIEW, Uri.parse("http://google.com"))
      startActivity(intent)
      ```
    * 링크를 띄울 수 있는 액션을 OS에서 받아 하단 Chooser 브라우저 목록 표시
- 면접 관련 질문
  + Activity 간 데이터 전달 방법?
    * Activity 간 데이터는 주로 Intent의 Extra를 통해 전달
    * 복잡한 데이터는 Serializable 또는 Parcelable 객체를 사용하며, 경우에 따라 Bundle을 통해 전달
  + Activity & Fragment 차이
    * | 항목      | Activity | Fragment              |
      | ------- | -------- | --------------------- |
      | 실행 단위   | 독립 실행 가능 | Activity 내부에서만 실행     |
      | 생명주기    | 독립적      | Activity 생명주기에 종속     |
      | 사용 목적   | 전체 화면 구성 | 부분 화면, 동적 UI 구성       |
      | 재사용성    | 낮음       | 높음                    |
      | Manager | 시스템이 관리  | `FragmentManager`가 관리 |
    * Fragment 사용 시 장점
      1. 메모리 효율성 높음 (View 만 교체 가능 → 가벼움)
      2. 화면 재사용 용이 (여러 Activity 에서 호출 가능)
    * Activity는 UI뿐 아니라 자체 Window와 Context를 포함하기에 시스템 차원에서 더 많은 초기화 비용이 발생.
    * 반면 Fragment는 Activity 위에서 동작하며, View와 Lifecycle만 별도로 관리하므로 생성 비용이 훨씬 가벼움
  + Activity 에서 메모리 누수 발생 상황?
    * "Activity는 View, Context 등을 포함하고 있어서, 생명주기와 무관한 참조가 남아 있으면 GC되지 않아 메모리 누수 발생할 수 있음
      1. 익명 클래스 Handler
      2. 전역 static 변수에 Context 저장
      3. 비동기 작업 (예: Retrofit, RxJava) 완료 전에 Activity가 종료됨
      4. ViewBinding을 onDestroy에서 해제하지 않음 (Fragment도 포함)
    * 해결 방법
      1. WeakReference 사용
      2. lifecycleScope, viewLifecycleOwner.lifecycleScope 활용
      3. onDestroy()에서 리소스 해제
  + Activity 화면 회전 시 상태 보존 방법?
    * 화면 회전 시 Activity는 재생성되므로, 
      1. **onSaveInstanceState()**를 사용해 데이터를 임시 저장, 
      2. **onCreate() 또는 onRestoreInstanceState()**에서 복원
    * ViewModel 사용하여, 구성 변경(Configuration Change) 시에도 데이터를 유지


---


## 4대 컴포넌트 BroadcastReceiver
- 정의
  + 시스템 또는 앱에서 발생하는 브로드캐스트 이벤트를 수신하고, 이에 대응하는 코드를 실행하는 컴포넌트
    * 안드로이드 시스템은 다양한 **이벤트(알람, 네트워크 변화, 부팅 완료 등)**를 브로드캐스트로 보냄
    * 앱도 커스텀 브로드캐스트를 만들어 보낼 수 있음
    
- 특징
  + | 항목    | 설명                                |
    | ----- | --------------------------------- |
    | 실행 방식 | 콜백 함수인 `onReceive()`가 호출되어 처리     |
    | 생명주기  | 화면 UI 없음. `onReceive()`만 실행되고 종료됨 |
    | 등록 방식 | 정적 등록 (Manifest), 동적 등록 (코드에서)    |
    | 목적    | 이벤트 감지 및 간단한 처리를 빠르게 수행           |

- 사용 방법
  + 정적 등록(Manifest)
    * ```xml
      <receiver android:name=".BootReceiver">
        <intent-filter>
           <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
      </receiver>
      ```
  + 동적 등록(코드 방식)
    * ```kotlin
      val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
          Log.d("Receiver", "Battery changed!")
        }
      }

      val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
      registerReceiver(receiver, filter)
      ```
      1. 앱이 실행 중일 때만 수신 가능
      2. 주로 Activity, Service 내에서 사용

- 주의사항
  + onReceive()는 UI 작업 금지 (짧고 빠른 처리만 수행해야 함)
  + 오래 걸리는 작업은 JobIntentService, WorkManager, ForegroundService 연계 필요
  + Android 8.0(Oreo) 이상에서는 Manifest 등록 제한이 많아 동적 등록 또는 백그라운드 제한 우회 필요

- 면접 관련 질문
  + Android 8.0(Oreo) 이후 백그라운드 앱의 정적 브로드캐스트 수신이 제한
    * 보안과 배터리 최적화
      1. Manifest 등록한 브로드캐스트 리시버는 일부 시스템 이벤트만 수신할 수 있게 되었고,
      2. 대부분 앱이 실행 중일 때 동적으로 등록해야 수신 가능
  + onReceive() 에서 긴 시간의 작업 실행 시?
    * 내부적으로는 앱 프로세스가 일시적으로 깨워진 상태에서 동작함
    * 긴 작업을 실행 시 **ANR(Application Not Responding)**이 발생하거나 시스템이 강제로 종료
  + BroadcastReceiver 대신 사용할 수 있는 대
    * Android 8.0 이상 JobScheduler, JobIntentService, WorkManager 같은 백그라운드 작업 매니저
    * LiveData, StateFlow, EventBus 방법 등으로 대체 가능
  + BroadcastReceiver vs Service
    * | 항목           | **BroadcastReceiver**                          | **Service**                                             |
      | ------------ | ---------------------------------------------- | ------------------------------------------------------- |
      | **목적**       | 이벤트 수신 후 짧은 작업 처리                              | 백그라운드에서 장시간 작업 수행                                       |
      | **트리거 방식**   | 외부 or 내부에서 **브로드캐스트**로 호출됨 (`sendBroadcast()`) | 명시적으로 **시작(`startService`)** 또는 바인딩(`bindService`)      |
      | **생명주기**     | `onReceive()` 단 하나, 실행 후 바로 종료                 | `onCreate()` → `onStartCommand()` → `onDestroy()` 등 존재  |
      | **UI와의 관계**  | UI 없음 / 사용자와 상호작용 X                            | UI 없음 / Activity와는 분리되어 있음                              |
      | **작업 시간 제약** | **매우 짧아야 함** (몇 초 이내) → 길면 ANR 발생              | 상대적으로 **장시간 작업 가능**                                     |
      | **실행 상태 유지** | 호출 후 자동 종료됨                                    | 명시적으로 종료되기 전까지 **계속 실행** 가능                             |
      | **멀티스레드 처리** | 자체 스레드 아님 (메인 스레드에서 동작)                        | 직접 스레드 생성 가능 (`HandlerThread`, `Coroutine`, `Thread` 등) |
      | **사용 사례**    | 네트워크 상태 변경 감지, 부팅 완료 감지, 앱 내 메시지 처리            | 파일 다운로드, 음악 재생, 알람 대기 등                                 |


---


## PendingIntent
- 정의
  + 다른 앱 또는 시스템이 지정된 작업(Intent) 대신 실행할 수 있도록 허가하는 객체
  + 알림(Notification), 알람(AlarmManager), 위젯(AppWidget), 브로드캐스트 예약 등에 쓰임
  > Intent 캡슐화하여 나중에 다른 컴포넌트(시스템 또는 외부 앱)가 실행할 수 있도록 위임하는 객체

- 필요한 이유?
  + 시스템이나 다른 앱이 내 앱의 Context 없이 특정 작업을 수행해야 할 때 사용
  + AlarmManager, NotificationManager 같은 시스템 서비스는 앱의 Intent를 직접 실행할 수 없음
  + 즉, 앱이 대신 실행해달라고 요청하는 형태로 감싸서 전달해야 함

- 주요 사용 케이스
  + | 상황                     | 사용 예                                |
    | ---------------------- | ----------------------------------- |
    | 알림 클릭 시 특정 Activity 열기 | `Notification`과 함께 사용               |
    | 특정 시간에 알림 보내기          | `AlarmManager`에서 예약 시 사용            |
    | 브로드캐스트 예약              | `PendingIntent.getBroadcast()` 사용   |
    | 서비스 시작                 | `PendingIntent.getService()` 사용     |
    | 앱 위젯에서 버튼 클릭 처리        | 위젯의 `RemoteViews`에 PendingIntent 연결 |
  + PendingIntent 메소드에는 getActivity(), getService() 도 있음

- 생성 방법
  + ```kotlin
    // Activity 실행
    val intent = Intent(context, MyActivity::class.java)
    val pendingIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_IMMUTABLE)

    // Broadcast 실행
    val intent = Intent(context, MyReceiver::class.java)
    val pendingIntent = PendingIntent.getBroadcast(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)

    // Service 실행
    val intent = Intent(context, MyService::class.java)
    val pendingIntent = PendingIntent.getService(context, 0, intent, PendingIntent.FLAG_IMMUTABLE)
    ```

- 주요 Flag
  + | 플래그                   | 설명                                            |
    | --------------------- | --------------------------------------------- |
    | `FLAG_IMMUTABLE`      | **Intent 내용 변경 불가** (보안 강화, Android 12+에서 필수) |
    | `FLAG_MUTABLE`        | 시스템이 Intent 내부를 수정할 수 있도록 허용                  |
    | `FLAG_UPDATE_CURRENT` | 같은 PendingIntent가 있으면 업데이트                    |
    | `FLAG_CANCEL_CURRENT` | 기존 PendingIntent 취소 후 새로 생성                   |
    | `FLAG_NO_CREATE`      | 이미 존재하는 경우에만 반환, 없으면 null                     |
    | `FLAG_ONE_SHOT`      |  한 번만 사용되고 나면 자동으로 소멸되도록 설정                  |
  
  + (| 또는 or) 연산으로 조합 가능
    * PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_ONE_SHOT

- 실무 팁
  + Android 12(API 31) 이상에서는 FLAG_IMMUTABLE 또는 FLAG_MUTABLE 중 하나를 반드시 명시해야 함
  + PendingIntent는 시스템에 캐싱되므로 중복 생성 주의
  + Request Code가 같고, Intent 내용이 같으면 같은 PendingIntent로 간주됨
  + 보안을 위해 불필요하게 mutable한 Intent는 지양

- 면접 관련 질문
  + PendingIntent란 무엇이며, 왜 필요한가요?
  + PendingIntent.FLAG_IMMUTABLE은 언제 사용하나요?
  + PendingIntent.getActivity()와 getBroadcast()의 차이는?


---


### Bundle
- 정의
  + key-value 쌍으로 데이터를 저장하고 전달하는 데 사용되는 핵심 클래스이며, 다양한 타입의 데이터를 담을 수 있습니다.

- Activity, Fragment 간 데이터 전달
  + Intent 객체에 Bundle을 담아 다른 Activity로 데이터를 전달할 수 있습니다.
  + Fragment를 생성할 때 arguments로 Bundle을 전달하여 초기 데이터를 넘겨줄 수 있습니다.

- Activity, Fragment 상태 저장 및 복원
  + 화면 회전과 같이 Activity나 Fragment가 시스템에 의해 재생성될때, onSaveInstanceState 콜백 메서드를 통해 현재 상태를 Bundle에 저장할 수 있습니다.
  + 이후 onCreate나 onRestoreInstanceState 또는 onCreateView(), onViewCreated()에서 저장된 Bundle을 받아 상태를 복원할 수 있습니다.

- Service와 데이터 교환
  + Service에 데이터를 전달하거나 Service로부터 결과를 받을 때 Bundle을 사용할 수 있습니다.

- 다양한 Android 컴포넌트 간 데이터 전달
  + BroadcastReceiver가 브로드캐스트를 수신할 때 Intent에 포함된 Bundle을 통해 데이터를 받을 수 있습니다.
  + ContentProvider를 통해 데이터를 전달할 때도 Bundle이 사용될 수 있습니다.

- Bundle의 주요 특징
  + key-value 저장소 : 문자열 키를 사용하여 다양한 타입의 값을 저장합니다.
  + 다양한 데이터 타입 지원 : String, int, long,boolean, float, double, char, byte, short와 같은 기본 타입뿐만 아니라,
                         CharSequence, Parcelable, Serializable, 배열, ArrayList 등 다양한 객체 타입을 저장할 수 있습니다.
  + Parcelable 인터페이스 : Bundle 자체는 Parcelable 인터페이스를 구현합니다. 이는 Bundle 객체가 프로세스 간 통신을 위해 효율적으로 직렬화될 수 있음을 의미합니다.
                         안드로이드 시스템은 Parcelable을 사용하여 객체를 Intent에 담아 다른 컴포넌트로 전달하거나, 프로세스 경계를 넘어 데이터를 전송합니다.
  + 데이터 크기 제한 : Bundle을 통해 전달되는 데이터의 크기에는 제한이 있습니다. 너무 큰 데이터를 Bundle에 담아 전달하려고 하면 TranscationTooLargeException이 발생할 수 있습니다.
                   일반적으로 10KB에서 1MB 미만으로 유지하는 것이 좋습니다. 

- Bundle 사용 시 주의 사항
  + null 체크 : Bundle에서 값을 꺼낼 때는 해당 키가 존재하지 않거나 값이 null일 수 있으므로, 항상 null체크를 하거나 기본값을 제공하는 함수(예 : getXX(key, defaultValue))를 사용 하는 것이 안전 합니다.
  + 키 관리 : Bundle에 데이터를 넣고 뺄 때 사용하는 키 문자열은 정확히 일치해야 합니다. 오타를 방지하기 위해 상수로 정의하여 사용하는 것이 좋습니다.
  + 데이터 타입 일치 : 데이터를 넣을 때 사용한 putType() 메서드와 꺼낼 때 사용한 getType()메서드의 데이터 타입이 일치 해야 합니다.
  + Serializable vs Parcelable : 복잡한 객체를 Bundle에 담을 때 Serializable이나 Parcelable을 사용할 수 있습니다.
    * Parcelable : 안드로이드에서 성능이 더 좋도록 특별히 설계되었으므로, 안드로이드 컴포넌트 간 데이터 전달에는 Parcelable 사용이 권장됩니다.
    * Serializable : 구현이 더 간단하지만 리플렉션을 사용하기 때문에 성능이 상대적으로 느립니다.


---


### 객체 직렬화
- **Serializable**
  - 객체를 **바이트 스트림으로 변환**하여 파일로 저장하거나 네트워크로 전송할 수 있게 하는 기능
  - 바이트 스트림을 역직렬화(Deserialization)를 통하여 원래 객채로 복원 가능

  - 직렬화 사용 이유
    - 저장 : 객체 상태를 파일에 저장하거나 DB에 저장
    - 전송 : 네트워크를 통해 객체를 다른 JVM으로 전달
    - 캐싱 : 객체를 메모리/디스크에 저장해 재사용
    - RPC : 원격 호출에서 객체 데이터를 주고 받을때 사용

  - 사용방법
    1. **Serializable** 인터페이스 구현
      ```java
          import java.io.Serializable;
       
          public class User implements Serializable {
              private static final long serialVersionUID = 1L;

              private String name;
              private int age;
           
              // 생성자, getter, setter 등
          }
      ```
    - Serializable은 마커 인터페이스로, 메서드가 하나도 없고 단지 직렬화 대상이라는 표시만 함

    2. 직렬화 예제 (저장)
      ```java
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.dat"));
          User user = new User("철수", 25);
          out.writeObject(user);
          out.close();
      ```
    - 역질렬화 예제 (복원)
      ```java
          ObjectInputStream in = new ObjectInputStream(new FileInputStream("user.dat"));
          User user = (User) in.readObject();
          in.close();
      ```
  - 주의할 점
    - serialVersionUID : 클래스 버전 관리를 위한 고유 ID (버전이 불일치할 경우 오류 발생 가능)
    - transient 키워드 : 직렬화에서 제외할 필드에 사용 (transient String password;)
    - static 필드 : 클래스에 속한 값이므로 직렬화 되지 않음
    - 직렬화 대상 객체의 모든 필드 : 직렬화가 가능해야 함 (안될경우 NotSerializableException 발생)

- **Parcelable**
  - 안드로이드에서 객체를 Intent나 Bundle로 전달할 때 사용되는 직렬화 방식
  ```kotlin
        val intent = Intent(this, DetailActivity::class.java)
        intent.putExtra("user", user)   // user는 객체
  ```
  - user가 일반 클래스면 안됨 -> Parcelable을 구현해야 넘기기 가능
  - 예제
    1. 데이터 클래스 만들기
    ```kotlin
        import android.os.Parcelable
        import kotlinx.parcelize.Parcelize
        
        @Parcelize
        data class User (val name: String, val age: Int) : Parcelable
    ```
    2. Intent에 담아서 전달
    ```kotlin
        val user = User("철수", 25)
        val intent = Intent(this, DetailActivity::class.java)
        
        intent.putExtra("user", user)
        startActivity(intent)
    ```
    3. 받은 쪽에서 꺼내기
    ```kotlin
        val user = intent.getParcelableExtra<User>("user")
    ```
- Parcelable vs Serializable

  | 항목  | Parcelable     | Serializable       |
            |-----|----------------|--------------------|
  | 속도  | 빠름 (메모리 직접 처리) | 느림 (리플렉션 기반)       |
  | 용도  | 안드로이드 앱 전용     | 자바 전반에 사용 가능       |
  | 코드량 | 많음 (직접 작성)     | 적음 (인터페이스만 붙히면 가능) |
  | 성능  | 고성능            | 저성능                |



---


### Application 클래스 역할
- 정의
  + 앱 프로세스가 시작될 때 생성되어, 앱 전역에서 사용 가능한 **싱글톤 컨텍스트(Context)**를 제공하는 클래스
  + onCreate()는 앱이 실행될 때 가장 먼저 호출
  + 앱이 완전히 종료되기 전까지 인스턴스는 살아있음
  + 전역 상태 관리, 초기화, 라이브러리 설정 등에 사용

- 주요 역할
  + | 역할                | 설명                                                             |
    | ----------------- | -------------------------------------------------------------- |
    | **전역 Context 제공** | 어떤 컴포넌트에서도 `applicationContext`로 접근 가능                         |
    | **앱 시작 시 초기화 작업** | DI 초기화, Logger 설정, SDK 초기화 등                                   |
    | **전역 상태/데이터 관리**  | 로그인 상태, 글로벌 설정, 싱글톤 객체 관리 등                                    |
    | **생명주기 감지**       | `registerActivityLifecycleCallbacks()`를 통해 Activity 생명주기 추적 가능 |
    | **공통 처리 추상화**     | 공통 에러 처리, 공통 폰트, 공통 테마 적용 등                                    |

- 사용 방법
  + MyApp 클래스 생성 및 Application 상속 
    * ```kotlin
      class MyApp : Application() {
         override fun onCreate() {
           super.onCreate()
           // 예: Timber 초기화, Hilt, Firebase 등
         }
      }
      ```
  + AndroidManifest.xml 선언
    * ```xml
      <application
         android:name=".MyApp" >
      </application>
      ```

- 주의사항
  + | 주의점                   | 설명                                    |
    | --------------------- | ------------------------------------- |
    | **메모리 누수 주의**         | Activity, Context 등 참조 저장 금지 (GC 안 됨) |
    | **성능 영향**             | `onCreate()`에서 너무 무거운 작업 금지           |
    | **Thread-safe 설계 필요** | 전역 변수 접근 시 동시성 문제 고려                  |

- 면접 관련 질문
  + Application 생성과 소멸 시점?
    * 앱 프로세스 시작 시 onCreate()가 호출되고, 앱이 완전히 종료될 때 소멸
  + Application과 Activity의 Context 차이?
    * Application은 앱 전역의 Context, Activity는 UI에 특화된 Context
    * Activity UI와 관련된 작업(ViewInflate, Theme 등)은 Application Context 에서 하면 안 됨
  + 전역 데이터 Application에 저장?
    * 간단한 설정 or 일시적 정보는 괜찮음
    * DB나 SharedPreferences가 필요한 복잡한 데이터 별도 관리 권장
    * 장기 보관은 ViewModel, Repository 등을 사용



---


### AndroidManifest.xml
- 정의 
  + Android 앱의 핵심 구성 파일 입니다.
  + 앱에 대한 필수 정보를 Android 빌드 도구, Android 운영체제 및 Google Play에 제공합니다.
  + 모든 Android 엡 프로젝트의 루트 디렉터리에 반드시 존재해야 합니다.

- 주요 역할 및 내용
  + 애플리케이션 패키지 이름
    * 앱의 고유 식별자 입니다.
    * Java 또는 Kotlin의 패키지 이름 규칙을 따릅니다
    * Google Play Store에서 앱을 식별하는 데 사용되며, 한번 게시된 후에는 변경할 수 없습니다.
    * <manifest> 태그의 package 속성에 정의됩니다.
    
  + 애플리케이션 컴포넌트 선언
    * 앱을 구성하는 모든 핵심 컴포넌트(Activity, Service, BroadcastReceiver, ContentProvider)를 선언해야 합니다.
    * 각 컴포넌트는 <activity>, <service>, <receiver>, <provider> 태그를 사용하여 선언됩니다.
    * 컴포넌트의 이름, 속성 기능 등을 정의합니다.
    * ```xml
      <application>
            <activity android:name=".MainActivity" android:exported="true">
                <intent-filter>
                    <action android:name="android.intent.action.MAIN" />
                    <category android:name="android.intent.category.LAUNCHER" />
                </intent-filter>
            </activity>
            <service android:name=".MyBackgroundService" />
            <receiver android:name=".MyBroadcastReceiver" android:exported="false">
                <intent-filter>
                    <action android:name="android.intent.action.BOOT_COMPLETED"/>
                </intent-filter>
            </receiver>
            <provider
                android:name=".MyContentProvider"
                android:authorities="com.example.myapp.provider"
                android:exported="true" />
        </application>
      ```
      
  + 권한 선언
    * 앱이 시스템의 보호된 부분이나 다른 앱의 데이터에 접근하기 위해 필요한 권한을 선언합니다.
    * <uses-permission> 태그를 사용하여 요청합니다.
    * 인터넷 접근, 카메라 사용, 위치 정보 접근 등
    * ```xml
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.CAMERA" />
      ```
      
  + 하드웨어 및 소프트웨어 기능 요구사항 선언
    * 앱이 정상적으로 작동하기 위해 필요한 하드웨어 또는 소프트웨어 기능을 명시합니다.
    * <users-feature>태그를 사용하여 선언 합니다.
    * 카메라 기능, 특정 센서 등
    * ```xml
      <uses-feature android:name="android.hardware.camera" android:required="true" />
      <uses-feature android:name="android.software.leanback" android:required="false" />
      ```
  
  + API 레벨 요구사항
    * 앱이 호환되는 최소 Android API 레벨, 타겟 API 레벨, 그리고 선택적으로 최대 API 레벨을 지정합니다.
    * <uses-sdk> 태그 내에 정의됩니다.
    * ```xml
      <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="33" />
      ```
      
  + 애플리케이션 메타데이터
    * 앱의 아이콘, 앱 이름, 테마 등 전역적인 애플리케이션 설정을 정의합니다.
    * <application> 태그의 속성으로 설정됩니다.
    * 추가적인 메타데이터를 <meta-data> 태그를 사용하여 키-값 쌍으로 제공할 수 있습니다.
    * 라이브러리나 특정 시스템 기능에 필요한 설정을 전달하는 데 사용됩니다.
    * ```xml
      <application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
            <meta-data
                android:name="com.google.android.geo.API_KEY"
                android:value="YOUR_API_KEY"/>
        </application>
      ```

  + 인텐트 필터
    * 컴포넌트가 어떤 종류의 인텐트에 반응할 수 있는지 시스템에 알립니다.
    * <intent-filter> 태그 내에 <action>, <category>, <data> 요소를 사용하여 정의합니다.
    
- 면접 관련 질문
  + taskAffinity, launchMode 설정이 앱 동작에 어떤 영향을 주나요?
    * launchMode는 Activity의 인스턴스 생성 방식, 
    * taskAffinity는 액티비티가 어떤 태스크에 소속될지를 결정 합니다.
    
  + Android App Bundle 사용시 Manifest에서 주의할 점은 무엇인가요?
    * AAB는 기능 모듈별로 APK를 생성하는 방식이므로 Manifest도 각 모듈 별로 존재할 수도 있고, 병합 규칙에 따라 최종 Manifest가 결정됩니다.

  + allowBackup, fullBackupContent 설정의 보안적 의미는 무엇인가요?
    * android:allowBackup="true"가 설정되면 사용자가 adb로 앱 데이터를 백업/복원할 수 있고,
    * 민감한 데이터가 포함된 앱에서는 보안 위험이 될 수 있습니다.




---



### Context
- 앱의 현재 상태와 환경에 대한 정보를 담고 있는 객체
- Context가 있어야 할 수 있는 일들 : 앱에서 무언가를 '실행'하거나 '접근'할 때 반드시 필요한 정보가 담겨있음
+ | 할 일        | 예시                                                     |
            |------------|--------------------------------------------------------|
  | 리소스 접근     | `getString(R.string.app_name)`                         |
  | 뷰 생성       | `LayoutInflater.from(context).inflate(...)`            |
  | 파일 읽기/쓰기   | `context.openFileInput("file.txt")`                    |
  | 토스트 띄우기    | `Toast.makeText(context, "Hello", Toast.LENGTH_SHORT)` |
  | 다른 액티비티 시작 | `context.startActivity(intent)`                        |
  | 서비스 시작     | `context.startService(...)`                            |

- Context 종류
+ | 종류                | 설명                                              |
              |-------------------|-------------------------------------------------|
  | Activity          | 화면(Activity)와 연결된 Context. 가장 많이 사용             |
  | Application       | 앱 전체에서 하나뿐인 전역 Context                          |
  | Service           | 백그라운드 작업할 때 쓰는 Context                          |
  | BroadcastReceiver | 브로드캐스트 수신 시 전달되는 Context                        |
  | ContextWrapper    | 다른 Context를 감싸는 클래스 (ex) `ContextThemeWrapper`) |

- `Application Context` VS `Activity Context`
    * | 구분        | `Application Context`             | `Activity Context`          |
      |-----------|-----------------------------------|-----------------------------|
      | 수명        | 앱이 켜져 있는 동안 계속 유지                 | 액티비티가 종료되면 사라짐              |
      | 어디서 사용?   | 토스트, DB, SharedPreferences 등 전역 기능 | 뷰, 다이얼로그, UI 등 화면 기반 기능     |
      | 메모리 누수 위험 | 거의 없음                             | 액티비티가 해제되지 않으면 메모리 누수 발생 가능 |
    * 예시
    ```kotlin
        // Application Context
        Toast.makeText(applicationContext, "전역 Toast", Toast.LENGTH_SHORT).show()
  
        // Activity Context
        val inflater = LayoutInflater.from(this)
    ```

- 주의 할 점
  - 메모리 누수 (Memory Leak) 주의
    * `Activity Context`를 싱글톤이나 오래 살아있는 객체에 저장하면 -> 액티비티가 메모리에 계속 남아있음 -> 메모리 누수 발생

- Context 없이 하지 못하는 것들
  + | 기능                                  | Context 필요               |
                  |-------------------------------------|--------------------------|
    | XML에서 View inflate                  | ✅ 필요                     |
    | 리소스 접근 (`getString`, `getDrawable`) | ✅ 필요   |
    | 화면 전환 (`startActivity`)             | ✅ 필요   |
    | SharedPreferences, DB, File         | ✅ 필요 |


---


### Weak/Strong Reference
- 개요
  + **WeakReference와 StrongReference**는 메모리 관리와 **Garbage Collection(GC)**에 있어서 매우 중요한 개념
  + 특히 Android 개발에서는 메모리 누수 방지, Context 참조 처리, 캐시 구현 등에서 필요함

- Strong Reference (강한 참조)
  + 일반적으로 사용하는 기본 참조 방식 default 
  + ```kotlin
    val activity = MainActivity()
    ```
  + 이 `Activity` 는 GC가 절대 수거하지 않음 (참조가 남아있는 한)
  + 메모리에서 명시적으로 null 처리하거나 범위에서 벗어나야 GC 대상이 됨
  + 즉, 객체 생명주기를 강하게 붙잡고 있는 구조
  
- Weak Reference (약한 참조)
  + 객체를 참조하되, GC가 수거해도 막지 않는 참조 방식
  + ```kotlin
    val activityRef = WeakReference(acitivty)
    ```
  + `WeakReference` 는 GC가 메모리가 부족할 경우 객체를 수거할 수 있도록 허용
  + 참조는 유지되지만, GC가 처리할 수 있도록 **약한 연결** 만 갖는 상태
  + 즉, 메모리 누수를 방지하거나 캐시를 유연하게 관리할 때 사용

- 비교
  + | 항목         | Strong Reference  | WeakReference                      |
    | ---------- | ----------------- | ---------------------------------- |
    | GC 대상 여부   | 참조가 존재하면 GC 대상 아님 | GC는 자유롭게 수거 가능                     |
    | 참조 유지      | 강하게 유지 (직접 소유)    | 약하게 유지 (GC 허용)                     |
    | 사용 예       | 대부분의 변수, 클래스 속성 등 | 캐시, Context 참조, 리스너, View 등        |
    | Android 활용 | 일반적인 코드           | Handler → Activity 참조 방지, LruCache |

- 실전 예제
  + 잘못된 사용 예제
    * ```kotlin
      class MyHandler(val activity: Activity) : Handler() {
          override fun handleMessage(msg: Message) {
              activity.doSomething()
          }
      }
      ```
    * Activity가 끝나도 Handler가 내부에서 강하게 참조 중이면 GC 불가 → 메모리 누수
    * ```kotlin
      class MyHandler(activity: Activity) : Handler() {
         private val activityRef = WeakReference(activity)

         override fun handleMessage(msg: Message) {
             val activity = activityRef.get()
             activity?.doSomething()
         }
      }
      ```
    * Activity가 끝나면 GC가 수거 가능하도록 WeakReference 로 해결

- 면접 관련 질문
  + WeakReference는 언제 사용하나요? 실무에서의 예시도 함께 설명해주세요.
    * WeakReference는 객체를 참조하면서도 GC가 해당 객체를 수거할 수 있도록 허용할 때 사용
    * Android에서는 Activity나 Context처럼 생명주기가 명확하고 무거운 객체를 오래 참조하면 메모리 누수 위험이 있음
    * Handler나 Listener 내부에서 Activity를 참조할 때 WeakReference를 사용
    * 오래 실행되는 타이머나 핸들러가 Activity를 직접 참조하면 화면이 닫힌 뒤에도 메모리가 회수되지 않기 때문에 WeakReference(activity)로 감싸서 GC가 수거할 수 있게 합니다."
  + WeakReference 객체에서 실제 객체를 사용할 때 주의해야 할 점은?
    * WeakReference.get()은 반환값이 nullable이기 때문에, 항상 null 체크 후 사용
    * 이 참조는 언제든 무효화될 수 있으므로, 장시간 신뢰 가능한 객체 상태가 필요하다면 별도의 강한 참조 유지가 필요할 수 있음
  + GC 캐시 정책 관련 참조 타입
    * Strong/Soft/Weak/Phantom 4가지의 참조타입이 존재
    * soft 는 캐시용으로 적합하며 메모리 부족할때만 수거 (LRU캐시처럼 동작)
    * phantom 객체가 finalize 된 이후 추적용으로 쓰임. 리소스 해제 후 처리 작업 추적 용도이나 복잡해서 거의 쓰이질 않음



---


### ANR(Application Not Responding)
- 정의
  + 안드로이드 앱에서 ANR이 발생하는 주된 이유는 앱의 UI 스레드가 너무 오랫동안 차단되어
    시스템이 입력 이벤트나 화면 그리기를 제때 처리하지 못하기 때문입니다.

- ANR이 발생하는 주요 조건 및 제한 시간
  + 입력 전달 타임아웃
    * 앱이 입력 이벤트에 5초 이내에 응답하지 않을 때 발생합니다.
    * 가장 흔한 ANR 유형이며, 사용자가 직접적으로 경험하기 때문에 매우 중요합니다.
  + 서비스 실행시간 초과
    * 앱에서 선언한 Service가 Service.onCreate(), Service.onStartCommand(), Service.onBind()와 같은 주요 생명주기 메서드를 몇 초이내 완료하지 못할때 발생합니다.
  + Service.startForeground() 호출 지연
    * 앱이 Context.startForegroundService()를 사용하여 포그라운드에서 새 서비스를 시작했지만, 해당 서비스가 5초이내 startForeground()를 호출하지 않은 경우 발생합니다.
  + 브로드캐스트 수신 시간 초과
    * BroadcastReceiver가 onReceive() 메서드 실행을 설정된 시간이내 완료 하지 못할때 발생합니다.
  + JobScheduler 상호작용 시간 초과
    * JobService가 JobService.onStartJob() 또는 JobService.onStopJob()에서 몇 초 이내 반환되지 않을 때 발생할 수 있습니다.
    * ANR 발생의 일반적인 원인 : ANR의 근본 원인은 대부분 메인 스레드에서 시간이 오래 걸리는 작업을 수행하기 때문입니다.
  + 메인 스레드에서의 느린 코드 실행
    * 네트워크 작업 : 메인 스레드에서 직접 네트워크 요청을 보내고 응답을 기다리는 경우 (예: HTTP 요청, 소켓 통신)
    * 파일 I/O 작업 : 메인 스레드에서 대용량 파일을 읽거나 쓰는 경우 (예: 데이터베이스 접근, SharedPreferences 읽기/쓰기, 파일 시스템 접근)
    * 복잡한 계산 또는 루프 : 메인 스레드에서 CPU를 많이 사용하는 복잡한 알고리즘이나 긴 루프를 실행하는 경우
    * 잘못된 UI 업데이트 : 너무 많은 UI 객체를 한 번에 그리거나, 비효율적인 방식으로 UI를 업데이트하는 경우
  + 잠금 경합
    * 메인 스레드가 다른 스레드가 점유하고 있는 잠금을 얻기 위해 대기하는 경우 발생합니다.
    * 만약 워커 스레드가 해당 잠금을 오랫동안 해제하지 않으면, 메인 스레드는 계속 차단되어 ANR로 이어질 수 있습니다.
    * 특히 메인 스레드가 워커 스레드의 작업 완료를 기다리는 동기화 코드에서 자주 발생합니다.
  + 교착 상태
    * 두 개 이상의 스레드가 서로가 가진 리소스를 기다리며 무한 대기 상태에 빠지는 경우입니다.
    * 만약 메인 스레드가 교착 상태에 포함되면 ANR이 발생합니다.
  + 느린 브로드캐스트 리시버 처리
    * BroadcastReceiver의 onReceive() 메서드는 짧은 시간 내에 완료되어야 합니다.
    * onReceive() 내에서 네트워크 요청, 파일 I/O, 복잡한 계산 등의 오래 걸리는 작업을 직접 수행하면 ANR이 발생할 수 있습니다.
    * 오래 걸리는 작업은 JobScheduler나 WorkManager 등을 사용하여 백그라운드 스레드로 옮겨야 합니다.
  + 시스템 서버 문제 또는 높은 기기 부하
    * 때로는 앱 자체의 문제보다는 시스템 전체의 상태 때문에 ANR이 발생할 수도 있습니다.
  + StrictMode 사용
    * 개발 중에 StrictMode를 활성화하여 메인 스레드에서 실수로 실행되는 디스크 I/O나 네트워크 작업을 감지할 수 있습니다.
  + 백그라운드 스레드 활용
    * 시간이 오래 걸리는 작업은 반드시 백그라운드 스레드로 옮겨야 합니다.
  + ANR 트레이스 분석
    * ANR이 발생하면 시스템은 /data/anr/ 디렉터리에 트레이스 파일을 생성합니다.
    * ANR 발생 시점의 모든 스레드 상태와 스택 트레이스가 포함되어 있어, 어떤 스레드가 무엇을 하고 있었는지, 특히 메인 스레드가 어디서 차단되었는지 분석하는 데 매우 중요합니다.
  + Perfetto 트레이스 사용
    * 시스템 전체의 성능을 분석할 수 있는 Perfetto 트레이스를 사용하여 앱의 메인 스레드가 실제로 실행 중인지, 아니면 다른 문제로 인해 차단되었는지 더 자세히 파악할 수 있습니다.
  + 코드 최적화
    * 비효율적인 알고리즘이나 UI 업데이트 로직을 개선합니다.
  + 잠금 관리 최적화
    * 잠금 사용을 최소화하고, 잠금 순서를 일관되게 유지하여 교착 상태를 방지하여, 방금 유지 시간을 최대한 짧게 가져갑니다.
    * ANR은 사용자 이탈의 주요 원인이므로, 앱의 응답성을 유지 하는 것은 매우 중요합니다.
    * 메인 스레드는 항상 가볍게 유지하고, 시간이 걸리는 작업은 백그라운드로 위임하는 습관을 들여야 합니다.



---


### 의존성 주입 (DI)
- 어떤 클래스가 다른 클래스의 기능에 의존하고 있을 경우, 그 다른 클래스가 의존성(Dependency)
```kotlin
    class Car {
        val engine = Engine()
    }
    // Car이 Engine에 의존함
```

- 의존성 주입 (DI)
  - 의존하는 객체를 클래스 내부에서 직접 만들지 않고 **외부에서 넣어주는 것**
  - ex)
    1. 이전 방식 (직접생성 - 나쁨)
    ```kotlin
        class Car {
            val engine = Engine()   // 직접 생성
        }   
    ```
    2. 주입 방식
    ```kotlin
        class Car (val engine : Engine)     // 밖에서 넣어줌 
        // Car은 Engine의 구체적인 생성 방법을 몰라도 됨
    ```
- 사용해야하는 이유
  + | 이유      | 설명                      |
                      |---------|-------------------------|
    | 유지보수 용이 | 클래스 간 연결이 약해져서 쉽게 수정 가능 |
    | 테스트 쉬움  | 테스트 시 가짜(Mock) 객체 주입 가능 |
    | 확장성     | 코드를 바꾸지 않고 기능 추가 가능     |
    | 결합도 낮춤  | 클래스 간의 결합도를 줄이고 재사용성 증가 |

- 안드로이드에서 DI가 중요한 이유 ➡ DI가 **객체 생성 책음을 관리**하고 코드 테스트를 쉽게 만들어줌
  - Context, ViewModel, Repository, UseCase 등 다양한 객체들이 서로 연결되어 있음
  - 수명 주기가 복잡하고 메모리 누수가 발생할 수 있음
  - 테스트가 어렵고 환경에 따라 로직이 바뀔 수 있음

- 안드로이드에서 DI 도구
  + | DI 프레임워크  | 설명                                      |
                          |-----------|-----------------------------------------|
    | Hilt      | 구글이 만든 공식 안드로이드 DI 도구 (Dagger 기반, 자동설정) |
    | Dagger2   | 매우 강력하지만 설정 복잡 (Hilt의 기반이 됨)            |
    | Koin      | 코틀린 친화적 DSL 기반 DI (간단하고 직관적)            |
    | Manual DI | 직접 생성자 주입/팩토리 작성 (작은 프로젝트에 적합)          |

- Hilt 예시
  ```kotlin
        // Repository 정의
        class UserRepository @Inject constructor() {
            fun getUser() : String = "철수"
        }     
    
        // ViewModel에서 주입받기
        @HiltViewModel
        class MainViewModel @Inject constructor (private val repository : UserRepository) : ViewModel() {
            val user = repository.getUser()
        }
  
        // Application 클래스 설정
        @HiltAndroidApp
        class MyApp : Application()
  ```


---


### WorkManager
- 정의
  + 백그라운드 작업을 안정적으로 실행하기 위한 Jetpack 라이브러리
  + 앱 종료 또는 디바이스 재부팅 시 작업 보장되어야 할 때 사용하는 핵심 컴포넌트

- 사용 시점
  + | 사용 예        | 설명                  |
    | ----------- | ------------------- |
    | 주기적 데이터 동기화 | 서버와 주기적으로 동기화       |
    | 파일 업로드/백업   | 사용자 몰라도 백그라운드에서 처리  |
    | 장기 실행 작업 예약 | 앱 종료/재부팅 후에도 보장     |
    | 네트워크 필요 작업  | Wi-Fi 상태 체크 후 실행 가능 |

- 특징
  + | 기능                                | 설명                     |
    | --------------------------------- | ---------------------- |
    | ✅ 작업 보장                           | 앱이 종료돼도, 재부팅돼도 실행됨     |
    | ✅ 제약 조건 지원                        | Wi-Fi, 충전 중일 때 등 설정 가능 |
    | ✅ JobScheduler, AlarmManager 등 통합 | OS 버전에 따라 자동 내부 분기     |
    | ✅ Chaining 지원                     | 작업 순서 정의 가능 (`then()`) |
    | ✅ Coroutine & RxJava 호환           | 코루틴 기반 작업 작성 가능        |

- 샘플 코드
  + 의존성 추가 필요
    * implementation "androidx.work:work-runtime-ktx:$work_version"
  + Work 클래스
    * ```kotlin
      import android.content.Context
      import android.util.Log
      import androidx.work.Worker
      import androidx.work.WorkerParameters
  
      class MyWorker(appContext: Context, params: WorkerParameters) : Worker(appContext, params) {
  
          override fun doWork(): Result {
             // 작업 수행
             Log.d(MyWorker::class.java.simpleName, "작업 실행됨!")
             return Result.success()
          }
      }
      ```
  + 사용 방법
    * ```kotlin
      // WorkRequest 생성 & 작업 예약
        val workRequest = OneTimeWorkRequestBuilder<MyWorker>().build()
        WorkManager.getInstance(this).enqueue(workRequest)

        // 반복 작업 (주기적) 최소 15분 이상 간격만 가능
        val periodicWork = PeriodicWorkRequestBuilder<MyWorker>(15, TimeUnit.MINUTES).build()
        WorkManager.getInstance(this)
            .enqueueUniquePeriodicWork("my_work", ExistingPeriodicWorkPolicy.KEEP, periodicWork)

        // 제약 조건 설정
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresCharging(true)
            .build()

        val work = OneTimeWorkRequestBuilder<MyWorker>()
            .setConstraints(constraints)
            .build()

        // 작업 체이닝
        val work1 = OneTimeWorkRequestBuilder<MyWorker>().build()
        val work2 = OneTimeWorkRequestBuilder<MyWorker>().build()

        WorkManager.getInstance(this)
            .beginWith(work1)
            .then(work2)
            .enqueue()
      ```

- 면접 관련 질문
  + WorkManager vs JobScheduler vs AlarmManager 차이
    * WorkManager는 내부적으로 플랫폼에 따라 JobScheduler나 AlarmManager를 선택해 사용
    * 앱 상태와 OS 버전 상관없이 동작을 보장함. 즉, 통합된 고수준 API
  + Worker, CoroutineWorker, RxWorker 차이?
    * Worker: 동기 방식
    * CoroutineWorker: 코루틴 기반
    * RxWorker: RxJava 기반



---



### DiffUtil
- **RecyclerView의 리스트 데이터가 변경되었을 때, 변경된 부분만 갱신**해주는 유틸리티
  1. 사용 이유
    + **기존 방식 (**`notifyDataSetChanged()`**)**
      ```kotlin
          adapter.notifyDataSetChanged()  // 리스트 전체를 다시 그림
      ```
      * 모든 아이템이 다시 그려짐 (비효율적)
      * 깜빡임, 스크롤 위치 초기화, 애니메이션 없음

    + **`DiffUtil` 방식**
       ```kotlin
           val diffResult = DiffUtil.calculateDiff(diffCallback)
           diffResult.dispatchUpdatesTo(adapter)
       ```
      * 변경된 항목만 새로 그림
      * 애니메이션 자동 처리
      * 리스트 스크롤 위치 유지
  2. 사용 시기
  + | 조건                            | DiffUtil 사용 권장 여부 |
          |-------------------------------|-------------------|
    | RecyclerView에서 데이터 리스트가 자주 바뀜 | ✅ 필요              |
    | 변경된 항목만 효율적으로 갱신하고 싶음         | ✅ 필요              |
    | J깜빡임 없는 UI와 부드러운 애니메이션 필요     | ✅ 필요              |
    | 정적 리스트 (변경 없음)                | ❌ 필요 없음           |

- 면접 예상 질문
  + DiffUtil이 무엇인가요?
    * RecyclerView의 데이터를 효율적으로 갱신하기 위한 유틸리티 클래스
    * 변경된 항목만 감지해 애니메이션과 함께 부분 업데이트 수행
    * `notifyDataSetChanged()` 대신 사용
  + DiffUtil 과 notifyDataSetChanged의 차이점이 무엇인가요?
    * `notifyDataSetChanged()`는 전체 리스트를 다시 그리므로 비효율적
    * DiffUtil은 변경된 항목만 계산해서 해당 위치에만 업데이트 호출 -> 성능 및 Ux 개선


---


### 안드로이드 8.0 이후 브로드캐스트 제한사항
- 개요
  + Android 8.0부터는 배터리 수명과 시스템 성능을 향상시키기 위해 백그라운드 실행에 대한 제한이 강화되었습니다.
  + Android 8.0 이전에는 앱이 매니페스트에 브로드캐스트 리시버를 등록해두면, 앱이 실행 중이지 않더라도 시스템이나 다른 앱에서 발생하는 다양한 브로드캐스트를 수신할 수 있었습니다.
    이는 편리했지만, 많은 앱이 특정 시스템 이벤트에 응답하기 위해 백그라운드에서 깨어나 리소스를 소모하는 원인이 되었습니다.
    이로 인해 배터리가 빠르게 소모되고 시스템 성능이 저하될 수 있었습니다. 안드로이드 8.0에서는 이러한 문제를 해결하기 위해 매니페스트에 등록된 리시버가 수신할 수 있는 브로드캐스트의 종류를 제한했습니다.

- 핵심 변경 사항
  + 대부분 암시적 브로드캐스트 수신 불가 : 앱이 Android 8.0 이상을 타겟팅하는 경우, 매니페스트에 등록된 리시버는 더 이상 대부분의 암시적 브로드캐스트를 수신할 수 없습니다.
  + 명시적 브로드캐스트는 계속 수신 가능 : 매니페스트에 등록된 리시버는 여전히 명시적 브로드캐스트를 수신할 수 있습니다.
  + 컨텍스트에 등록된 리시버는 계속 동작 : Context.registerReceiver()를 사용하여 동적으로 등록된 리시버는 앱이 활성 상태일 때 암시적 브로드캐스트를 포함한 모든 브로드캐스트를 계속 수신할 수 있습니다.

- 자세한 설명
  + 제한의 대상
    * 브로드캐스트 Android 8.0에서 주로 제한되는 것은 암시적 브로드캐스트입니다.
  + 예시
    * android.net.conn.CONNECTIVITY_CHANGE (CONNECTIVITY_ACTION): 네트워크 연결 상태가 변경될 때 발생하는 브로드캐스트입니다.
      Android 7.0 (API 레벨 24)부터 이미 매니페스트 등록이 제한되었지만, Android 8.0에서는 이 제한이 더 광범위하게 적용됩니다.
    * android.hardware.action.NEW_PICTURE (ACTION_NEW_PICTURE): 새로운 사진이 찍혔을 때 발생하는 브로드캐스트입니다. 이 역시 Android 7.0부터 제한되었습니다.

- 예외적으로 허용되는 암시적 브로드캐스트
  + 모든 암시적 브로드캐스트가 제한되는 것은 아닙니다. 시스템 운영에 중요하거나, 앱이 특정 기능을 수행하기 위해 반드시 필요한 일부 브로드캐스트는 여전히 매니페스트에 등록된 리시버를 통해 수신할 수 있습니다.
  + 예외 브로드캐스트의 예
    * ACTION_BOOT_COMPLETED: 기기가 부팅을 완료했을 때 발생합니다.
    * ACTION_LOCKED_BOOT_COMPLETED: 직접 부팅(Direct Boot) 모드에서 사용자가 기기를 잠금 해제하기 전에 발생합니다.
    * SMS/MMS 관련 브로드캐스트 (예: SMS_RECEIVED_ACTION, WAP_PUSH_RECEIVED_ACTION)

- 컨텍스트 등록 리시버의 역할 강화
  + 매니페스트 등록 리시버의 기능이 제한됨에 따라, 컨텍스트 등록 리시버의 중요성이 커졌습니다.
  + Context.registerReceiver(): Activity나 Service와 같은 컴포넌트의 컨텍스트를 사용하여 코드 내에서 동적으로 브로드캐스트 리시버를 등록합니다.
  + 생명주기: 컨텍스트 등록 리시버는 해당 컨텍스트가 유효한 동안에만 브로드캐스트를 수신합니다.
    예를 들어, Activity 컨텍스트에 등록된 리시버는 Activity가 활성 상태일 때만 동작합니다.
  + 장점: 앱이 사용자에게 보여지거나 활발하게 사용될 때만 브로드캐스트를 수신하므로, 불필요한 백그라운드 작업을 줄일 수 있습니다.

- 앱 타겟 API 레벨의 중요성
  + 이러한 브로드캐스트 제한은 기본적으로 앱이 Android 8.0 (API 레벨 26) 이상을 타겟팅할 때 적용됩니다.

- 결론
  + Android 8.0의 브로드캐스트 제한은 앱의 백그라운드 동작을 최적화하여 사용자 경험을 향상시키기 위한 중요한 변경 사항입니다.
  + 개발자는 이러한 제한 사항을 이해하고, 앱이 최신 Android 버전에서도 안정적으로 동작하도록 JobScheduler, WorkManager, 컨텍스트 등록 리시버, 포그라운드 서비스 등의 대안을 적극적으로 활용해야 합니다.
  + 이는 배터리 효율성을 높이고 시스템 리소스 사용을 최적화하는 데 기여합니다.

- 면접 예상 질문
  + Android 8.0이상에서 브로드캐스트 관련 제한사항은 무엇인가요?
    * 암시적 브로드캐스트의 정적 등록제한
    * 백그라운드 앱은 브로드캐스트 수신 제한
    * 명시적 브로드캐스트나 런타임 등록 권장
    * 배터리 최적화 및 성능 향상을 위한 정책 변화
  + 암시적 브로드캐스트와 명시적 브로드캐스트의 차이점은?
    * 암시적 : 특정 수신자를 지정하지 않고 Intent 필터에 따라 시스템이 전달
    * 명시적 : setComponent() 또는 setPackage()로 특정 리시버 지정
    * Android 8.0부터는 명시적 브로드캐스트 권장


---


### SharedPreferences & DataStore
- SharedPreferences
  + 작은 양의 key-value 형태 데이터를 로컬에 저장할 수 있도록 제공되는 기본 저장 API
  + 주로 로그인 정보, 앱 설정, 토글 상태 등 간단한 데이터를 저장하는데 사용
  + 핵심특징
    * XML 파일 기반 저장
    * 동기적 or제한적인 비동기 방식 (apply())
    * 앱 재시작 후에도 데이터 유지됨 (영속성)
    * Thread-safe 보장 없기에 다중 접근 시 위험
    * 복잡한 데이터 구조 저장에 부적합
  + Sample Code
    * ```kotlin
      // 데이터 쓰기
      val prefs = context.getSharedPreferences("my_prefs", Context.MODE_PRIVATE)
      prefs.edit {
        putString("userName", "james")
        .putBoolean("isLogin", true)
      }
      
      // 데이터 읽기
      val username = prefs.getString("userName", null)
      val isLoggedIn = prefs.getBoolean("isLogin", false)
      ```

- DataStore
  + Jetpack에서 제공하는 최신 데이터 저장 솔루션으로, 비동기 처리와 타입 안전성을 갖춘 Key-Value 저장 방식
  + SharedPreferences의 단점을 보완하고, Flow 및 Coroutine을 기반으로 설계
  + 핵심 특징
    * 코루틴 기반 비동기 저장
    * Flow 통한 실시간 데이터를 감지 및 수집
    * 두 종류 제공
      1. `Preferences DataStore`: Key-Value 저장
      2. `Proto DataStroe`: 구조화된 커스텀 객체 저장 (protobuf 기반)
    * 스레드 안전 + ANR 방지
    * JetPack 공식 권장 방식
  + Sample Code
    * 의존성 추가 `implementation("androidx.datastore:datastore-preferences:1.0.0")`
    * ```kotlin
      class UserPreferencesRepository(private val context: Context) {
        // 초기 설정
        private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "user_settings")

        // 키 정의
        object UserPreferencesKeys {
          val USERNAME = stringPreferencesKey("userName")
          val IS_LOGIN = booleanPreferencesKey("isLogin")
        }

        val userData: Flow<Pair<String?, Boolean>> = context.dataStore.data
          .map { prefs ->
            val username = prefs[UserPreferencesKeys.USERNAME]
            val isLoggedIn = prefs[UserPreferencesKeys.IS_LOGIN] ?: false
            username to isLoggedIn
        }

        suspend fun saveUser(username: String, isLoggedIn: Boolean) {
          context.dataStore.edit { prefs ->
            prefs[UserPreferencesKeys.USERNAME] = username
            prefs[UserPreferencesKeys.IS_LOGIN] = isLoggedIn
          }
        }

        suspend fun clearUser() {
          context.dataStore.edit { prefs ->
            prefs.clear()
          }
        }
      }
      ```

- 비교
  + | 항목            | SharedPreferences | DataStore             |
    | ------------- | ----------------- | --------------------- |
    | 저장 구조         | Key-Value         | Key-Value / Proto 구조  |
    | 저장 방식         | 동기 / 제한적 비동기      | 완전 비동기 + Coroutine 기반 |
    | 데이터 접근 방식     | 직접 접근             | Flow (Reactive)       |
    | 스레드 안전성       | ❌ 보장 안됨           | ✅ 보장됨                 |
    | 타입 안정성        | ❌                 | ✅ (특히 ProtoDataStore) |
    | 사용 용도         | 간단 설정값, 플래그 등     | 동기화 필요, 실시간 반응 등      |
    | Jetpack 권장 여부 | ❌ (구식 API)        | ✅ (공식 권장 방식)          |

- 면접 관련 질문
  + SharedPreferences와 DataStore 중 어떤 것을 사용?
    * 프로젝트에 Coroutine과 Flow를 사용하고 있고, 데이터 안정성과 실시간 반응성이 중요하다면 DataStore가 적합
    * SharedPreferences는 마이그레이션 전 레거시 앱에서 빠르게 구현할 때는 유용하나 유지보수와 확장성 측면에서 DataStore가 더 좋음


---


### okHttp와 Retrofit 차이
- 정의
  + Android 및 Java 애플리케이션에서 네트워킹을 처리하는 데 널리 사용되는 라이브러리이지만, 서로 다른 계층에서 작동하며 다른 목적을 가지고 있습니다.

- OkHttp : HTTP 클라이언트
  + 역할
    * OkHttp는 HTTP 및 HTTP/2 클라이언트 입니다.
    * 실제 네트워크 요청을 만들고 응답을 받는 저수준(low-level) 라이브러리입니다.
  + 주요 기능
    * HTTP GET, POST, PUT, DELETE 등 다양한 요청 메서드 지원
    * 요청/응답 헤더 및 본문 처리
    * 연결 풀링을 통한 성능 최적화
    * Gzip 압축을 통한 데이터 크기 감소
    * 응답 캐싱
    * 타임아웃 설정
    * TSL/SSL 보안 연결 지원
    * HTTP/2 및 SPDY 프로토콜 지원
    * 자동 재시도 및 리디렉션 처리
    * WebSocket 지원
  + 사용 방식 : 개발자가 직접 Request 객체를 만들고, OkHttpClient를 사용하여 이를 실행하고, Response 객체를 받아 처리해야 합니다.
    ```kotlin
    // OkHttp 사용 예시
    val client = OkHttpClient()

    val request = Request.Builder()
        .url("https://api.example.com/data")
        .header("Authorization", "Bearer your_token")
        .build()

    client.newCall(request).enqueue(object : Callback {
        override fun onFailure(call: Call, e: IOException) {
            // 오류 처리
        }

        override fun onResponse(call: Call, response: Response) {
            if (response.isSuccessful) {
                val responseBody = response.body?.string()
                // 응답 처리 (예: JSON 파싱)
            } else {
                // 서버 오류 처리
            }
        }
    })
    ```
  + 장점
    * 매우 효율적이고 강력한 HTTP 통신 기능 제공
    * 세밀한 요청/응답 제어 가능
    * 다양한 네트워크 최적화 기능 내장
  + 단점
    * 상대적으로 저수준 API이므로, 반복적인 코드 작성이 필요할 수 있음
    * API 엔드포인트가 많아질수록 코드 관리가 복잡해질 수 있음

- Retrofit : 타입-세이프 HTTP 클라이언트
  + 역할
    * Retrofit은 타입-세이프한 REST 클라이언트입니다. 즉, HTTP API를 Java/Kotiln 인터페이스로 변환하여 사용하기 쉽게 만들어주는 고수준 라이브러리입니다.
  + 기반
    * Retrofit은 내부적인 OkHttp를 HTTP 클라이언트로 사용합니다. 즉 Retrofit은 OkHttp 위에 구축된 추상화 계층입니다.
  + 주요 기능
    * 인터페이스 기반 API 정의 : 어노테이션을 사용하여 HTTP 요청을 인터페이스 메서드로 매핑합니다.
    * 자동 직렬화/역직렬화 : 요청 본문과 응답 본문을 Java/Kotlin 객체로 자동으로 변환해줍니다.
    * 동기/비동기 요청 지원 : Kotlin 코루틴, RxJava, Call 객체를 통한 비동기 처리 지원
    * URL 파라미터, 쿼리 파라미터, 요청 본문 등을 어노테이션으로 쉽게처리
    * 멀티파트 요청지원
  + 사용 방식 : 개발자는 API 엔드포인트를 정의하는 인터페이스를 만들고, Retrofit 빌더를 사용하여 해당 인터페이스의 구현체를 생성합니다.
    ```kotlin
    // Retrofit 인터페이스 정의 예시
    interface ApiService {
        @GET("data")
        suspend fun getData(@Header("Authorization") token: String): MyData // MyData는 데이터 모델 클래스
    }

    // Retrofit 사용 예시
    val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .client(OkHttpClient()) // 선택 사항: 커스텀 OkHttpClient 설정 가능
        .addConverterFactory(GsonConverterFactory.create()) // JSON <-> 객체 변환기
        .build()

    val apiService = retrofit.create(ApiService::class.java)

    // 코루틴 내에서 호출
    viewModelScope.launch {
        try {
            val myData = apiService.getData("Bearer your_token")
            // 성공 처리
        } catch (e: Exception) {
            // 오류 처리
        }
    }
    ```
  + 장점
    * API 호출 코드를 매우 간결하고 직관적으로 작성 가능
    * 타입-세이프 : 컴파일 시점에 API 정의 오류를 발견할 수 있음
    * JSON/XML 등의 데이터 변환을 자동으로 처리하여 보일러플레이트 코드 감소
    * OkHttp의 강력한 네트워킹 기능을 그대로 활용 가능
  + 단점
    * OkHttp에 대한 의존성을 가짐
    * 매우 세밀한 저수준 제어가 필요한 경우 OkHttp를 직접 사용하는 것보다 유연성이 떨어질 수 있음
- OkHttp와 Retrofit의 관계 및 차이점 요약
  + | 특징       | OkHttp                    | Retrofit              |
    |----------|---------------------------|-----------------------|
    | 계층       | 저수준 HTTP 클라이언트            | 고수준 REST 클라이언트        | 
    | 주요목적     | HTTP 요청/응답처리, 네트워크 최적화    | API엔드포인트를 인터페이스로 추상화  |
    | 사용방식     | Request.Response 객체 직접 처리 | 어노테이션 기반 인터페이스 정의, 자동 객체 매핑 |
    | 코드 복잡도   | 상대적으로 높음                  | 상대적으로 낮음 |
    | 데이터 변환   | 수동처리                      | 자동처리 |
    | 의존성      | 독립성                       | OkHttp에 의존 |




---



### Handler, Looper, MessageQueue
+ | 구성요소         | 역할      | 설명                                   |
            |--------------|---------|--------------------------------------|
  | Handler      | 메시지 전송자 | 작업에서 메시지를 생성에서 Looper에게 전달           |
  | MessageQueue | 메시지 저장소 | 전송된 메시지를 시간순으로 저장 (FIFO 큐)           |
  | Looper       | 메시지 처리기 | MessageQueue를 무한히 돌면서 메시지를 하나씩 꺼내 실행 |

+ 전체 구조 흐름
  ```markdown
    - Thread
        └── Looper
              └── MessageQueue
                       ↑
                   Handler.post()
  ```
  1. `Handler`가 `Message` 또는 `Runnable`을 생성하여 `MessageQueue`에 넣음
  2. `Looper`는 `MessageQueue`를 계속 돌면서 메시지를 꺼냄
  3. 해당 메시지를 처리 (`Handler.handleMessage()` 또는 `Runnable.run()` 호출)

+ Handler
  * **Runnable** 또는 **Message**를 전달할 수 있는 인터페이스
  * `post(Runnable)` 또는 `sendMessage(Messgae)` 사용
  * `handlerMessage(Message msg)`를 override 하여 메시지 처리 가능

+ Looper
  * `Thread`에 붙어서 해당 쓰레드가 이벤트 루프를 돌도록 만듬
  * 기본적으로 UI 메인 쓰레드에는 `Looper.getMainLooper()`가 존재
  * 새 쓰레드에서 쓰고 싶다면 `Looper.prepare()` -> `Looper.loop()` 호출 필요

+ MessageQueue
  * 메시지들이 `enqueueMessage()`로 들어와 저장됨
  * `next()` 메서드로 메시지를 하나씩 꺼냄
  * `Looper.loop()`이 이 `next()`를 무한 반복

+ 코드 예시
  * 메인 쓰레드에서 Handler 사용
    ```kotlin
        val handler = Handler(Looper.getMainLooper())
    
        handler.post {
            // UI 쓰레드에서 실행
            textView.text = "작업 완료"
        }
    ```
    - `post()`는 내부적으로 Message를 만들어 MessageQueue에 등록
    - Main Looper는 이걸 꺼내서 실행
  
  * 새 Thread에서 Handler 만들기
    ```kotlin
        class MyThread : Thread() {
            lateinit var handler : Handler
    
            override fun run() {
                Looper.prepare()    // Looper를 이 Thread에 바인딩
                handler = Handler(Looper.myLooper()!!) {
                    println("새 쓰레드에서 메시지 처리 됨")
                    true
                }
                Looper.loop()   // 메시지 루프 시작 
            }
        }
    
        val t = MyThread()
        t.start()
    
        // 나중에 다른 쓰레드에서 호출
        t.handler.post {
            println(" 이 코드는 MyThread 안에서 실행됨")
        }
    ```

  * 딜레이, 반복 작업 가능
    ```kotlin
        val handler = Handler(Looper.getMainLooper())
        val runnable = object : Runnable {
            override fun run() {
                println("1초마다 반복")
                handler.postDelayed(this, 1000)
            }
        }
        handler.post(runnable)
    ```
    - `postDelayed()`로 딜레이 실행 가능
    - `removeCallbacks(runnable)`로 중단 가능

+ Handler 직접 써야하는 이유
  * | 상황                   | 이유                                    |
              |----------------------|---------------------------------------|
    | 다른 쓰레드에서 UI를 바꿔야 할 떄 | UI는 오직 메인 쓰레드에서만 수정 가능 -> Handler로 전달 |
    | 쓰레드 간 작업 분리          | 백그라운드 쓰레드에서 계산하고 -> 메인 쓰레드에서 결과 반영    |
    | 일정 시간 후 작업 실행        | `postDelayed`로 지연 실행 가능               |
    | 주기적 작업 처리            | `Runnable` 재전송 방식으로 반복 구현 가능          |

+ 질문
  + Handler란 무엇인가요?
    - `Message` 또는 `Runnable`을 다른 스레드로 전달하고, Looper를 통해 큐에 등록된 작업을 실행하게 해주는 클래스.
    - UI 쓰레드로 작업을 전달하거나 딜레이/반복 작업에도 사용
  + Looper란 무엇인가요? 왜 필요한가요?
    - 해당 쓰레드에 연결되어 있는 이벤트 루프 처리기
    - `MessageQueue`를 반복해서 확인하며 메시지를 하나씩 꺼내서 실행
    - `UI 쓰레드`에는 기본적으로 `Looper`가 존재
  + Handler는 어떤 쓰레드에서 동작하나요?
    - 생성 시 지정한 Looper에 따라 동작하므로 메인 쓰레드에서 실행
    - 직접 만든 쓰레드에 사용하려면 Looper.prepare()로 초기화 해야 함
  + Handler 대신 Coroutines를 써도 되지 않나요?
    - Coroutine은 Handler보다 추상화된 방식으로 스레드 전환을 처리 가능
    - `withContext(Dispatchers.Main)`으로 메인쓰레드 전환이 가능하며, **코드 가독성과 에러 처리 측면에서 유리**
    - 하지만 내부적으로 Dispatchers는 `Handler`를 기반으로 동작하는 경우가 많다



---



### Foreground vs Background
1. Foreground
  - 사용자가 **화면에서 직접 보고있고, 터치하거나 상호작용 중**인 앱 상태
  - 앱의 Activitiy 가 화면에 보여지고 있고, 사용자와 직접 상호작용 중일 때를 의미
  ```text
    onCreate() -> onStart() -> onResume() -> [Foreground 상태]
  ```
  - ex)
    1. 유튜브에서 영상 보기
    2. 카카오톡 채팅 화면을 열어둠
    3. 메모 앱에서 글 쓰기
  
  - 특징
    - 시스템 자원을 자유롭게 사용 가능 (CPU, 네트워크, 위치, 센서 등)
    - 사용자의 주의가 앱 집중되어 있어야 하므로 **빠른 응답이 요구**됨
    - 이 상태에서는 백그라운드의 제한이 적용되지 않음

2. Background
  - 앱이 실행 중이지만, **화면에 보이지 않는 상태**
  - 사용자가 다른 앱을 보고 있거나, 홈 화면으로 나간 경우 등
  ```text
    Foreground() -> onPause() -> onStop() ->[Background 상태]    
  ```
  - ex)
    1. 인스타그램을 보다가 홈 버튼 누름
    2. 전화가 와서 앱이 가려짐
    3. 다른 앱으로 전환 됨
  
  - 특징
    - 시스템 자원을 **제한적으로 사용 가능**
    - Android 8.0(API 26) 이후부터 백그라운드 작업에 제한 강화
    - 백그라운드에서 일정시간 지나면 앱이 종료될 수 있음
    - 서비스 또는 브로드캐스트를 사용할때도 제한이 있음

3. Foreground Service
  - 앱이 백그라운드에 있어도 계속 실행되어야 할 작업을 위해 실행되어야 할 서비스 종류
  - 알림(Notification)을 항상 보여줘야 함

  - 사용 이유
    - 일반 서비스는 백그라운드에서 쉽게 종료될 수 있음
    - Foreground Service는 시스템이 우선순위를 높게 잡고, 종료시키지 않음

  - ex)
    - 음악 앱에서 백그라운드 재생 중
    - 네이버 지도에서 실시간 길 안내
    - 피트니스 앱에서 운동시간 측정
    
  - 구현흐름
    ```kotlin
        val notification = NotificationCompat.Builder(this, "channel_id")
            .setContentTitle("서비스 실행 중")
            .build()

        startForeground(1, notification)  // 반드시 알림과 함께 실행       
    ```
    
  - Lifecycle 순서
    * | 시나리오    | 상태변화                                                |
                    |---------|-----------------------------------------------------|
      | 앱 실행    | onCreate() -> onStart() -> onResume() -> Foreground |
      | 홈 버튼 누름 | onPause() -> onStop() -> Background                 |
      | 앱 다시 열기 | onRestart() -> onStart() -> onResume()              |
      | 앱 종료    | onDestory()                                         |

- 시스템 제한사항 (Android 8.0 이후)
  1. ⚠️ Background 실행 제한
    - 오래된 방식의 Background Service 실행이 제한 됨
    - 해결방법 : Foreground Service, JobScheduler, WorkManager 사용
  
  2. 🔐 Background 위치 권한
    - 백그라운드 GPS, BLE 같은 민감한 자원 접근 시 권한 강화
    - Android 10 부터는 ACCESS_BACKGROUND_LOCATION 별도 요청 필요

- 앱 상태 구분하는 기준
  * | 구분           | 설명                                            |
                        |--------------|-----------------------------------------------|
    | Foreground 앱 | 사용자가 현재 보고있는 앱. 가장 높은 우순위                     |
    | Visible 앱    | 포그라운드는 아니지만 현재 화면에 보이고 있는 상태 ( ex) Dialog 등 ) |
    | Background 앱 | 사용자에게 보이지 않지만 실행 중인 앱                         |
    | Cached 앱     | 사용하진 않지만 나중을 위해 메모리에 저장된 앱                    |


---


### Interceptor
- 정의
  + retrofit2는 안드로이드에서 가장 많이 쓰이는 HTTP 클라이언트 라이브러리이며, 내부적으로 OkHttp를 사용하는데 OkHttp의 핵심 기능 중 하나가 바로 **Interceptor**이고 기능을 그대로 사용
  + Interceptor는 모든 네트워크 요청/응답을 **가로채서 가공하거나 검사할 수 있는 훅(Hook)** 이자 인터페이스**

- 특징
  + | 특징                        | 설명 |
    |-----------------------------|------|
    | 요청/응답 모두 개입 가능     | 헤더 추가, 응답 처리 등 |
    | 체인 구조                   | 여러 Interceptor를 연결 가능 (`chain.proceed()`) |
    | 순차 실행                  | 등록된 순서대로 실행됨 |
    | 인증 처리 용이              | JWT, OAuth 토큰 등 자동 삽입 가능 |
    | 로그, 디버깅에도 활용 가능   | OkHttpLoggingInterceptor |
    | 캐싱, 리트라이 커스터마이징 가능 | 고급 제어 가능 |

- 기본 구조
  + ```kotlin
    class AuthInterceptor(private val tokenProvider: () -> String) : Interceptor {
      override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
            .newBuilder()
            .addHeader("Authorization", "Bearer ${tokenProvider()}")
            .build()
        return chain.proceed(request)
      }
    }
    ```
  + 이 코드는 모든 요청에 Authorization 헤더를 자동으로 추가

- 샘플 코드
  + Retrofit + Interceptor 설정
    * ```kotlin
      val client = OkHttpClient.Builder()
        .addInterceptor(AuthInterceptor { getAccessToken() })
        .addInterceptor(HttpLoggingInterceptor().apply {
          level = HttpLoggingInterceptor.Level.BODY
        })
        .build()

      val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com")
        .client(client)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
      ```
  + 토큰 만료 시 자동 갱신
    * ```kotlin
      class TokenRefreshInterceptor(...) : Interceptor {
        override fun intercept(chain: Interceptor.Chain): Response {
            val request = chain.request()
            val response = chain.proceed(request)

            if (response.code == 401) {
                synchronized(this) {
                    val newToken = refreshAccessToken()
                    val newRequest = request.newBuilder()
                        .header("Authorization", "Bearer $newToken")
                        .build()
                    return chain.proceed(newRequest)
                } 
            }

           return response
        }
      }
      ```
    * ️만료된 토큰을 자동으로 갱신할 수 있음
    * 모든 API 호출이 통합된 방식으로 보안 처리됨

- 결론
  + | 용도          | Interceptor 활용 예                     |
    | ----------- | ------------------------------------ |
    | 인증 토큰 자동 추가 | 헤더에 Authorization 삽입                 |
    | 요청 공통 헤더    | User-Agent, Locale, App-Version 등 삽입 |
    | 응답 가공       | 응답 코드에 따른 처리                         |
    | 로그 기록       | `HttpLoggingInterceptor` 사용          |
    | 토큰 갱신       | `401` 응답 시 자동 재요청                    |
  + Interceptor는 네트워크의 관문이다. 요청과 응답 모두를 지켜보며 수정, 검사, 자동화할 수 있다. Retrofit을 쓰면 반드시 함께 써야 할 고급 툴

- 면접 관련 질문
  + Interceptor는 언제, 왜 사용하는가요?
    * Interceptor는 네트워크 요청이나 응답을 가로채서 공통 작업을 처리하거나 수정할 수 있도록 해주는 구조입니다.
      주로 다음과 같은 경우에 사용합니다:
      1. 모든 요청에 공통 헤더 추가 (예: Authorization, App-Version 등)
      2. 인증 토큰이 만료되었을 때 자동으로 갱신 처리
      3. API 요청/응답 로깅 및 디버깅
      4. 캐시 정책, 네트워크 리트라이 제어 등
    * 이런 공통 처리를 각 API마다 반복하지 않고 Interceptor로 한 곳에서 제어할 수 있어서, 유지보수성이 뛰어납니다.
  + addInterceptor()와 addNetworkInterceptor()의 차이는 무엇인가요?
    * addInterceptor()는 전체 요청/응답을 가로챌 수 있는 애플리케이션 레벨의 인터셉터입니다.
      1. 요청이 캐시에서 처리되든 네트워크로 가든 모두 동작합니다.
    * addNetworkInterceptor()는 오직 네트워크를 통해 실제 서버와 통신할 때만 동작합니다.
      1. 캐시에서 응답이 반환될 경우, 호출되지 않습니다.
    * 실무에서는 대부분 addInterceptor()를 사용하며, 네트워크 상세 조작이 필요할 때만 addNetworkInterceptor()를 사용합니다.
  + 토큰 인증이 필요한 앱에서 Interceptor를 어떻게 활용해봤나요?
    * Interceptor를 활용해서 모든 API 요청에 자동으로 Bearer 토큰을 추가했습니다.
    * 또한, 서버에서 401 응답을 받았을 때는 Interceptor 내에서 토큰을 자동 갱신하고 원래 요청을 재시도하는 로직도 구현했습니다.
    * 이 구조 덕분에 API 호출부에서는 인증 처리 코드를 신경 쓸 필요가 없었고, 보안 로직을 중앙 집중화할 수 있었습니다.
  + Interceptor에서 무한 루프나 요청 중복 문제가 발생할 수 있는 경우는 언제인가요?
    * Interceptor에서 401 처리 후 토큰을 갱신하고 요청을 다시 보낼 때,
      → 재시도 로직이 반복적으로 호출되면 무한 루프가 발생할 수 있습니다.
    * 이걸 방지하려면 다음과 같은 방어 로직을 넣습니다:
      1. 새로 보낸 요청에는 토큰 갱신 플래그를 붙이거나,
      2. 재귀 요청 횟수를 제한하거나,
      3. chain.request()의 URL이나 Tag로 이미 재시도했는지 확인
    * 이런 방어 코드를 통해 네트워크 폭주나 앱 크래시를 막을 수 있습니다.
  + Retrofit과 OkHttp의 관계는 무엇인가요?
    * Retrofit은 고수준 HTTP 클라이언트이며, 내부적으로 OkHttp를 기반으로 동작합니다.
      1. 요청 전송, 응답 수신, 캐싱, 연결 풀링 등 낮은 수준의 네트워크 처리는 모두 OkHttp가 수행하고,
      2. Retrofit은 이를 래핑해서 인터페이스 기반 API 호출, Gson 파싱, 코루틴 연동 같은 기능을 제공합니다.
    * Interceptor는 Retrofit이 아니라 OkHttp의 기능이며, Retrofit에서 OkHttpClient를 설정해줄 때 함께 구성합니다.


---


### Notification
- 개요
  + 안드로이드 알림은 앱이 사용자에게 중요하거나 시기적절한 정보를 제공하는 방법 중 하나입니다.
  + 앱이 현재 실행 중이지 않거나 백그라운드에 있을 때도 사용자에게 알림을 표시하여 상호작용을 유도할 수 있습니다.

- 알림의 목적 및 중요성
  + 시기적절한 정보 제공 : 사용자에게 새로운 메시지, 예정된 이벤트, 다운로드 완료 등과 같은 중요한 정보를 즉시 알립니다.
  + 사용자 참여 유도 : 알림을 통해 사용자가 앱을 다시 실행하거나 특정 작업을 수행하도록 유도할 수 있습니다.
  + 백그라운드 작업 알림 : 음악 재생, 파일 다운로드, 위치 추적과 같은 백그라운드 작업을 사용자에게 알리고 제어할 수 있는 인터페이스를 제공합니다.

- 알림의 기본 구성 요소
  + 알림은 시스템 템플릿에 의해 디자인되며, 앱은 템플릿의 각 부분에 콘텐츠를 정의합니다.

- 알림 채널 - Android 8.0 이상 필수
  + 안드로이드 8.0부터 모든 알림은 반드시 채널을 통해 게시되어야합니다.
  + 채널 생성 : 앱에서 보내는 알림의 종류별로 채널을 만들어야 합니다.
  + 사용자 제어 : 사용자는 시스템 설정에서 각 채널별로 알림 동작을 직접 제어할 수 있습니다.
  + 중요도 : 채널을 만들 때 중요도를 설정해야 하며, 이는 알림이 사용자에게 얼마나 방해가 될지를 결정합니다.

- 알림 생성 및 표시 단계
  + AndroidX Core 라이브러리 추가 : NotificationCompat API를 사용하기 위해 build.gradle 파일에 의존성을 추가합니다
  + 런타임 권한 요청 : Android 13 이상을 타겟팅하는 앱은 알림을 게시하기 위해 POST_NOTIFCATIONS 런타임 권한을 사용자로부터 받아야합니다.
  + 알림 콘텐츠 설정 : NotificationCompat.Builder를 사용하여 알림의 내용을 구성합니다.
  + 알림 채널 생성 및 중요도 설정 : 위에서 설명한 대로 채널을 생성하고 NotificationManager에 등록합니다.
  + 알림의 탭 동작 설정 : 사용자가 알림을 탭했을때 실행될 작업을 PeddingIntant를 사용하여 설정합니다.
  + 알림 표시 : NotificationManagerCompat를 사용하여 알림을 시스템에 게시합니다.

- 알림의 추가 기능
  + 확장 가능한 알림 : 더 많은 정보를 표시하기 위해 알림을 확장할 수 있습니다.
  + 알림 작업 : 알림에 직접 버튼을 추가하여 사용자가 앱을 열지 않고도 빠른 작업을 수행할 수 있도록 합니다.
  + 진행률 표기시 : 파일 다운로드와 같이 시간이 오래걸리는 작업의 진행 상태를 알림에 표시할 수 있습니다.
  + 알림 그룹 : 여러 알림을 하나의 그룹으로 묶어 표시할 수 있습니다.


---


### GSON & Moshi
- 직렬화(Serialization)
  + `직렬화`는 객체를 JSON, XML 등 문자열 형태로 변환하는 과정
  + `역직렬화(Deserialization)`는 문자열(JSON 등)을 다시 객체로 복원하는 과정
  + Android 앱에서는 API 통신 시 데이터를 객체 <-> JSON으로 변환할 일이 많은데 이럴 때 주로 쓰이는 것이 `Gson`, `Moshi` 같은 JSON 파서다.

- 공통점
  + | 항목 | 설명 |
    |------|------|
    | JSON <-> 객체 매핑 | 자동 지원 (Serialize/Deserialize) |
    | 어노테이션 지원 | `@SerializedName`, `@Json` 등 |
    | Null-safe 처리 가능 | 옵션 또는 어댑터로 제어 가능 |
    | Kotlin + Java 호환 | 모두 가능 |
    | Retrofit 연동 가능 | `converter-gson`, `converter-moshi`|

- 차이점
  + | 항목           | **Gson**                           | **Moshi**                           |
    |----------------|------------------------------------|-------------------------------------|
    | 개발 주체       | Google                             | Square (Retrofit 만든 회사)         |
    | Kotlin 지원     | 제한적 (Kotlin에 최적화 X)         | 우수함 (`KotlinJsonAdapterFactory`) |
    | 속도/성능       | 느린 편                            | 빠르고 가벼움                        |
    | Null 처리       | 더 관대함                          | 더 엄격함                            |
    | 타입 어댑터     | 복잡함 (`TypeToken` 필요)          | 심플하고 타입 안전함                |
    | 확장성          | 풍부한 기능, 오래된 레거시 지원    | 모던 설계, 가볍고 간결함            |
    | 빌드 사이즈     | 크다                               | 작다                                 |

- 샘플 코드 비교
  + GSON
    * ```kotlin
      data class User(
        @SerializedName("user_name") val name: String,
        val age: Int
      )

      val gson = Gson()
      val json = """{"user_name": "Lee", "age": 28}"""
      val user = gson.fromJson(json, User::class.java)
      val jsonOut = gson.toJson(user)
      ```
  + Moshi
    * ```kotlin
      @JsonClass(generateAdapter = true)
      data class User(
        @Json(name = "user_name") val name: String,
        val age: Int
      )

      val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .build()

      val adapter = moshi.adapter(User::class.java)
      val user = adapter.fromJson("""{"user_name": "Lee", "age": 28}""")
      val jsonOut = adapter.toJson(user)
      ```

- 선택 기준
  + | 상황                          | 추천                              |
    | --------------------------- | ------------------------------- |
    | Kotlin 프로젝트 (특히 data class) | ✅ Moshi                         |
    | 레거시 Java 프로젝트               | ✅ Gson                          |
    | 가볍고 빠른 파싱이 중요한 경우           | ✅ Moshi                         |
    | 기존 Gson 기반 코드 유지            | ✅ Gson (단, 필요시 Moshi 마이그레이션 고려) |


---


### View & ViewGroup
- 개요
  + 안드로이드 UI를 구성하는 가장 기본적인 빌딩 블록은 View와 ViewGroup입니다. 이 둘의 관계와 역할을 이해하는 것은 Android UI 개발의 핵심입니다.

- View
  + 개요
    * View클래스는 사용자 인터페이스의 기본 구성요소입니다.
    * 화면에서 사용자와 상호작용할 수 있는 모든 것의 기본 클래스입니다.
    * View는 화면의 특정 직사각형 영역을 차지하며, 해당 영역의 그리기와 이벤트 처리를 담당합니다.
  + 주요 특징 및 역할
    * 화면의 직사각형 영역 차지: 모든 View는 화면에 자신만의 공간을 가집니다. 이 공간의 크기와 위치는 레이아웃 과정에서 결정됩니다.
    * 그리기 (Drawing): View는 자신의 모양과 내용을 화면에 그립니다. onDraw(Canvas canvas) 메서드를 오버라이드하여 커스텀 그리기를 구현할 수 있습니다. 시스템은 필요할 때 이 메서드를 호출하여 View를 다시 그리도록 요청합니다.
    * 이벤트 처리 (Event Handling): 사용자의 터치, 키 입력 등과 같은 이벤트를 감지하고 반응합니다. onTouchEvent(MotionEvent event), onClickListener, onKeyListener 등을 통해 이벤트를 처리합니다.
    * 속성 (Properties): View는 다양한 속성을 가집니다. (예: ID, 배경색, 패딩, 가시성 등). 이러한 속성은 XML 레이아웃 파일이나 Kotlin/Java 코드에서 설정할 수 있습니다.
    * 측정 (Measuring): onMeasure(int widthMeasureSpec, int heightMeasureSpec) 메서드를 통해 자신의 크기를 결정합니다. 부모 ViewGroup으로부터 크기 제약 조건을 전달받아 자신의 콘텐츠와 제약 조건에 맞는 크기를 계산합니다.
    * 레이아웃 (Layout): onLayout(boolean changed, int left, int top, int right, int bottom) 메서드를 통해 화면 내 자신의 위치와 크기를 최종적으로 할당받습니다. 이 과정은 부모 ViewGroup에 의해 관리됩니다.

- ViewGroup
  + 개요
    * ViewGroup은 View 클래스와 특별한 서브클래스입니다.
    * ViewGroup의 주요 역할은 다른 View 객체들을 담는 컨테이너 역할을 하는 것입니다.
    * 즉, ViewGroup은 UI 요소들을 그룹화하고 이들의 레이아웃을 관리합니다.
  + 주요 특징 및 역할
    * 자식 View 포함 (Container): 여러 개의 View 또는 다른 ViewGroup을 자식으로 가질 수 있습니다. 이를 통해 복잡한 UI 계층 구조를 만들 수 있습니다.
    * 레이아웃 관리 (Layout Management): ViewGroup은 자식 View들의 크기를 측정하고 화면에 배치하는 방법을 결정합니다. 각 ViewGroup의 서브클래스는 자신만의 레이아웃 정책을 가집니다.
    * 이벤트 전달 (Event Dispatching): ViewGroup은 터치 이벤트와 같은 사용자 이벤트를 받으면, 해당 이벤트가 발생한 위치에 있는 적절한 자식 View에게 이벤트를 전달할지, 아니면 자신이 직접 처리할지를 결정합니다. (onInterceptTouchEvent(), dispatchTouchEvent())
    * 계층 구조 (Hierarchy): ViewGroup은 다른 ViewGroup을 자식으로 가질 수 있으므로, UI 요소들은 트리(tree)와 같은 계층 구조를 형성합니다. 이 구조의 최상위에는 일반적으로 DecorView라는 특별한 ViewGroup이 있으며, 그 아래에 개발자가 정의한 레이아웃이 배치됩니다.

- View와 ViewGroup의 관계 요약
  + View는 개별 UI 요소입니다.
  + ViewGroup은 이러한 View들을 담는 컨테이너이며, 이들의 배치 방식을 결정합니다.
  + ViewGroup은 View를 상속받으므로, ViewGroup 자체도 하나의 View로 취급될 수 있습니다. 
  + UI는 View와 ViewGroup의 계층적인 트리 구조로 구성됩니다. ViewGroup은 다른 View나 ViewGroup을 자식으로 가질 수 있습니다.




---



### jar, arr, apk
1. JAR (Java ARchive)
  + 개념
    * Java 클래스 파일과 메타데이터를 압축한 아카이브
    * Java 기반 라이브러리를 패키징 할때 사용
  + 구성
    * .class 파일 (컴파일 된 Java 바이트코드)
    * META_-INF/MANIFEST.MF (메타 정보)
    * 리소스 파일 (.properties, .xml 등 포함 가능, 하지만 Android 리소스는 불가능)
  + 사용처
    * Java 코드만 있는 라이브러리
    * ex) Gson, OkHttp 등 일부 라이브러리 jar 버전
  + ❌ 한계
    * res/, AndroidManifest.xml 같은 안드로이드 전용 리소스 포함 불가

2. APK (Android Package)
  + 개념
    * .apk 는 Android 애플리케이션 패키지 파일
    * 안드로이드 앱을 기기에 설치할때 사용
  + 구성
    * class.dex (Dalvik / ART 바이트코드)
    * res/, assets/, AndroidManifest.xml
    * META-INF/ (서명 정보)
    * 네이티브 라이브러리 (lib/)
  + 사용처
    * 사용자 단말기에 설치되는 최종 결과물
    * GooglePlay에 업로드 하는 파일
  + 빌드 흐름
    ```text
      .java/.kt → .class → .dex → APK
    ```
    
3. ARR (Android ARchive)
  + 개념
    * .arr은 Android 라이브러리 모듈을 위한 패키지 파일
    * JAR의 안드로이드 버전
  + 구성
    * classes.jar (컴파일 된 java/kotlin 코드)
    * res/ (레이아웃, 이미지 등 리소스 파일)
    * AndroidManifest.xml
    * assets/, jni/, proguard.txt 등
  + 사용처
    * Android 모듈을 라이브러리로 분리할 때
    * 다른 프로젝트에서 재사용 가능
    * ex) UI 컴포넌트, SDK 제공용 라이브러리
  + 예시
    ```kotlin
      dependencies {
        implementation("com.example:mylibrary:1.0.0") // 내부적으로 aar 사용
      }
    ```
    
- 차이점 요약
  + | 항목            | jar        | aar                 | apk          |
                |---------------|------------|---------------------|--------------|
    | 목적            | Java 라이브러리 | Android 라이브러리       | Androi 앱 패키지 |
    | 포함 가능한 것      | Java 코드만   | 코드 + 리소스 + Manifest | 전체 앱 리소스     |
    | 설치 가능 여부      | ❌          | ❌                   | ✅            |
    | 리소스 포함 여부     | ❌          | ✅                   | ✅            |
    | Manifest 포함 여부 | ❌          | ✅                   | ✅            |
    | 사용 대상         | 공통 로직      | Android 모듈 공유       | 최종 사용자 기기    |

- 사용 시기 예시 
  + | 상황                              | 사용 파일 |
        |---------------------------------|-------|
    | Gson, Apache Commons 등 Java 유틸리티 | JAR   |
    | 버튼, 커스텀뷰, SDK 등 Android 재사용 모듈   | ARR   |
    | 앱 배포 (스토어 등록, 설치)    | APK   |

- 면접 질문
  + JAR, ARR, APK의 차이가 무엇일까요?
    * JAR : Java 아카이브로, 컴파일된 .class 파일과 메타정보만 포함하며, 안드로이드 리소스는 포함할 수 없습니다. 주로 Java 라이브러리에서 사용됩니다.
    * AAR: Android 전용 아카이브로, classes.jar 외에도 res/, AndroidManifest.xml, assets/ 등을 포함할 수 있어 Android 라이브러리 모듈 배포에 사용됩니다.
    * APK: Android Package로, 실제 앱을 설치할 수 있는 최종 패키지입니다. 실행 파일(.dex), 리소스, 서명 정보까지 모두 포함합니다.
  + AAR 파일은 어떤 경우에 사용하나요?
    * 안드로이드 앱에서 재사용 가능한 모듈을 분리해 라이브러리로 만들고, 이를 다른 프로젝트에 배포하거나 의존성으로 추가할 때 사용됩니다.
    * ex) 커스텀 버튼 UI, 공통 ViewModel, 인증 SDK 등을 AAR로 만들어 다른 앱에서 활용할 수 있습니다.
  + AAR과 JAR의 주요 차이점은 무엇인가요?
    * AAR은 Android 라이브러리 전용 형식으로, 코드뿐 아니라 리소스(res/), AndroidManifest, assets 등을 포함할 수 있습니다.
    * 반면 JAR은 Java 코드만 포함하므로, Android 앱에서 UI 구성이나 리소스를 사용하는 경우 적합하지 않습니다.