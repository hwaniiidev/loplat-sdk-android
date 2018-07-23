﻿# Plengi SDK

## Installation

### How to import

#### 1. repository 추가 하기
- In your top-level project build.gradle, add
	
	```gradle
	maven { url "http://maven.loplat.com/artifactory/plengi"}
	```

	as repositories under allprojects -> repositoreies.
	For example,

	```gradle	
	allprojects {
		repositories {
	        jcenter()
			mavenCentral()
			maven { url "http://maven.loplat.com/artifactory/plengi"}
	        google()
		}
	}
	```

#### 2. loplat SDK dependency 추가 하기
 - 앱 build.gradle (Gradle 3.0 이상)

	```gradle	
	implementation 'com.loplat:placeengine:[version]'
	```
 
- 앱 build.gradle (Gradle 3.0 미만)

	```gradle
	compile 'com.loplat:placeengine:[version]'
	```

## Contents
1. SDK Specification
2. SDK Setup
	- 계정 만들기 
	- Permission 등록
	- Receiver & Service 등록
	- Library 적용하기
	- Constraints 
3. SDK 기능
4. SDK 초기화
	- PlengiListener 생성
	- Plengi Instance 생성 및 EventListener 등록
	-  Plengi Init
5. SDK 구동하기
	- Plengi 모드 설정
	- WiFi 스캔 주기 설정
	- Gravity 연동하기
	- Start/Stop
	- 장소 인식 결과
6. API
	- 현재 위치 확인하기
	- 현재 사용자 상태(Move/Stay) 확인하기 
	- 현재 장소 정보 가져오기

### 1. SDK Specification

##### SDK 지원 버전
* minSdkversion 14
* targetSdkversion 26


### 2. SDK Setup

#### 계정 만들기  
* Plengi SDK를 사용하기 위해서는 clientid와 clientsecret 필요합니다.  
	 > * clientid & clientsecret: loplat server로 접근하기 위한 ID와 PW  
* test를 원하시는 분은 clientid: loplatdemo, clientsecret: loplatdemokey 사용하세요.  
*  정식 clientid와 clientsecret을 원하는 분은 아래에 기입 된 메일 주소로 연락 바랍니다. 
 
#### Permission
* SDK를 적용하면 하기 권한이 자동으로 추가됩니다.  
	```xml
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />  
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
	<uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    ```

	* ACCESS_FINE_LOCATION: GPS를 이용하여 현재 위치의 위도와 경도 값을 획득할 수 있는 권한  
	* ACCESS_COARSE_LOCATION: WiFi 혹은 Network를 이용하여 현재 위치의 위도와 경도 값을 획득할 수 있는 권한  
	* ACCESS_NETWORK_STATE: 네트워크 상태를 확인할 수 있는 권한  
	* ACCESS_WIFI_STATE / CHANGE_WIFI_STATE: 주변 WiFi AP들을 스캔하기 위한 권한
	* INTERNET: 인터넷을 사용할 수 있는 권한
	* RECEIVE_BOOT_COMPLETED: 핸드폰 부팅되는 과정을 브로드캐스팅하기 위한 권한

#### Constraints

* Android OS Marshmallow 버전 부터 WiFi Scan시 아래와 같은 위치 권한이 필요합니다.
	```xml
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
	```
* Plengi SDK 동작하기 위해서 위치 권한, GPS 상태, WiFi scan 가능 여부 등을 확인을 위한 작업이 필요합니다.
	
	* 확인 방법 및 설정은 sample코드에 구현된 checkWiFiScanCondition() (in MainActivity.java) 참고 바랍니다.
	* **[참고] Marshmallow 부터 위치 권한 허용 & GPS on 상태에서만 WiFi scan 결과값 획득이 가능합니다.**
	* Marshmallow 버전 부터 위치 권한은 Dangerous Permission으로 구분 되어 권한 획득을 위한 코드가 필요합니다.
	* 권한 설정과 관련하여 좀 더 자세한 사항은 Android Developer를 참고 바랍니다. [Android Developer](http://developer.android.com/intl/ko/training/permissions/requesting.html)

#### Gradle 설정 및 Library 적용

##### Android Oreo에서의 동작을 위한 gradle 설정 및 library 적용
- **SDK v1.8.8 이상 version 적용할 경우 해당**
- compileSdkVersion 설정 

	```gradle
	android {
		compileSdkVersion 26
		...
	}
	```
- android support library 적용
	- 아래의 예시 처럼 appcompat v7 라이브러리 version 26이상으로 적용
	- **주의**: appcompat v7의 version이 26이상이 아닌 경우에는 SDK 동작 중 error가 발생하여 **앱 동작이 중지 될 수 있으니**, 기존의 적용된 라이브러리 버전을 확인 후 업그레이드 하시길 바랍니다.
		```gradle
		compile 'com.android.support:appcompat-v7:26.1.0'
		```

##### Gravity 사용을 위한 Google Play Services library 적용하기

- Gravity를 사용하기 위해서 build.gradle의 dependency에 아래와 같이 google play service library 적용이 필요합니다.

	```gradle
	compile 'com.google.android.gms:play-services-ads:11.8.0'
	```
##### Retrofit 및 GSON library 적용하기

- loplat SDK 1.7.10 이상 버전 부터 위치 확인 요청시 서버와의 통신을 위해 Retrofit 및 GSON library 사용합니다. Retrofit 및 GSON 라이브러리 적용을 위해서  Android Studio의 build.gradle에 다음과 같이 추가합니다.

	```gradle
	compile 'com.squareup.retrofit2:retrofit:2.3.0'
	compile 'com.squareup.retrofit2:converter-gson:2.3.0'
	compile 'com.squareup.okhttp3:okhttp:3.8.1'
	```
	
- 참고: proguard를 사용할 시에는 아래와 같이 proguard 설정을 추가해야 합니다.

	```proguard
	-dontwarn okio.**
	-dontwarn javax.annotation.**
	-keepclasseswithmembers class * {
			@retrofit2.http.* <methods>;
	}
	```

### 3. SDK 기능

#### Recognize a place
* Plengi.getInstance(Context Context).refreshPlace() : 주변 WiFi AP들을 탐색하여, 서버에게 현재 위치 정보를 요청
* PlengiEventListener: listen()를 통해 Plengi 서버로 부터 받은 결과를 수신하여 PlengiBraodcastReceiver로 송신
* PlengiBroadcastReciver: PlengiEventLinstener로 부터 받은 결과 처리
 
#### Place Event
* Plengi.getInstance(Context Context).start()/ Plengi.getInstance(Context Context).stop()을 통해서 모니터링을 on/off
* PlengiEventListener와 PlengiBroadcastReceiver를 통해 Event 결과를 처리

#### Stay or Move
* Plengi.getInstance(Context Context).getCurrentPlaceStatus()를 통해 사용자가 현재 이동 중인지 한 장소에 머물고 있는지 확인 가능

#### Get a current place information
* Plengi.getInstance(Context context).getCurrentPlaceInfo()를 통해 사용자가 방문 중인 장소 정보 불러오기


### 4. SDK 초기화

#### 1. PlengiListener 생성 
* PlengiListener 인터페이스를 구현합니다.
	- loplat서버로 부터 받은 모든 asynchronous Result는 모두 해당 리스너를 통해 전달됩니다.
	- PLACE(Recognize a place), PLACE_EVENT(Enter/Leave/Nearby, Recognizer mode), PLACE_TRACKING(Enter/Leave/Nearby, Tracker mode) 등의 Event에 따른 결과를 작성합니다. (LoplatPlengiListener.Java 참조 바람)

- 예시코드

```java
public class LoplatPlengiListener implements PlengiListener {
	@Override
	public void listen(PlengiResponse response) {
		if(response.result == PlengiResponse.Result.SUCCESS) {
			if(response.type == PlengiResponse.ResponseType.PLACE_EVENT 
				|| response.type == PlengiResponse.ResponseType.PLACE_ADV_TRACKING 
				|| response.type == PlengiResponse.ResponseType.PLACE_TRACKING) { //BACKGROUND
				if (response.place != null) {
					// response.place 값이 null이 아닌 경우만 response.placeEvent(ENTER/LEAVE/NEARBY) 값을 사용
					int event = response.placeEvent;
					if (event == PlengiResponse.PlaceEvent.ENTER) { 
						// 사용자가 장소에 들어 왔을 때
					} else if (event == PlengiResponse.PlaceEvent.LEAVE) {
						// 사용자가 가장 최근에 머문 장소를 떠났을 때
					} else if (event == PlengiResponse.PlaceEvent.NEARBY) {
						// 사용자가 장소 주변(Nearby)을 방문 했을 때
					} 
				}
				if (reponse.area != null) {
					// 상권이 인식 되었을 때
				}
				if (response.complex != null) {
					// 복합몰이 인식 되었을 때
				}
			} 
		} else {
			// 위치 획득 실패 및 에러
			// response.errorReason 위치 획득 실패 혹은 에러 이유가 포함 되어 있음
			// errorReason -> Location Acquisition Fail(위치 획득 실패), Network Fail(네트워크 연결 실패), Not Allowed Client(잘못된 client id, passwrod 입력), invalid scan results
			if (response.result == PlengiResponse.Result.FAIL) {
				// response.errorReason 확인
			} else if (response.result == PlengiResponse.Result.ERROR_CLOUD_ACCESS) {
				// response.errorReason 확인
			}
		}
	}
}
```
#### 2. Plengi instance 생성 및 EventListener 등록
- Application class 상속 받아 Plengi class 생성합니다. (LoplatSampleApplication.java 참고 바람)
	- Plengi instance를 생성한 후, 1번에서 생성한 Listener를 등록합니다.
	
#### 3. Plengi init (LoplatSampleApplication.java 참고 바람)

```loplat_caution
   Application class onCreate()에 init(SDK 초기화)과 setListener(리스너 등록)을 
   선언하지 않으면 SDK가 동작하지 않습니다.
```

- 사용자의 매장/장소 방문을 모니터링하기 위해 Plengi Engine을 초기화합니다.
- **생성한 Application class(2번 항목 참조)에서 Plengi init을 다음과 같이 선언을 합니다.** 

	```java
	Plengi.getInstance(this).init(clientId,clientSecret,uniqueuserId);  
	```
	
- init을 위해 clientid, clientsecret, uniqueUserId를 인자값으로 전달해주셔야합니다.  
	* clientid & clientsecret: loplat server로 접근하기 위한 ID와 PW입니다.  
	* 정식 id와 secret을 원하는 분은 아래에 기입 된 메일 주소로 연락 바랍니다.  
	* uniqueUserId: App에서 사용자를 식별하기 위한 ID입니다. (ex, 광고id,id,....,etc.)
		* 이메일, 폰번호와 같이 **개인정보와 관련된 정보**는 전달하지 않도록 주의해주시기 바랍니다.

* 예시코드
```java
public class ModeApplication extends Application {
	Plengi mPlengi = null;
	private static ModeApplication instance;
	public static Context getContext() {
		return instance;
	}  
 
    @Override
    public void onCreate() {
	    super.onCreate();
	    instance = this;
	    mPlengi = Plengi.getInstance(this);
	    mPlengi.setListener(new LoplatPlengiListener());
	    mPlengi.init("[CLIENT_ID]", "[CLIENT_SECRET]", "[UNIQUE_USER_ID]");
	} 
}
```

### 5. SDK 구동하기
#### 1 . Plengi 모드 설정  
* 매장/장소 방문을 확인하기 위한 모니터링 모드를 선택합니다.    
* 사용자의 매장/장소 방문을 확인하기 위하여 아래와 같은 3가지 모드를 제공하고 있습니다.  
	* Recognizer Mode: 일정시간동안(5분이상) 한 장소에 머무를 경우 사용자의 위치를 확인합니다.
	* Tracker Mode: 사용자의 위치를 일정주기마다 확인합니다.
	* Advanced Tracker Mode: Android에서 제공하는 awareness api를 이용하여 효율적으로 사용자의 위치를 확인합니다.
		* Awareness API는 Google Play Service를 제공하는 API이며, 여러가지 센서를 통해 Applicaition이 사용자의 상황(ex. 걷고있음, 이어폰을 연결함 등)을 인지할 수 있도록 제공하는 API
		* Awareness API는 Android System Framework을 이용하여  효율적으로 배터리 소모량을 관리함 
		* **참고사항**: Android System Framework에서 App의 배터리 소모를 관리 하므로 App의 배터리 소모량이 확인되지 않음
		* 하루 한도 사용량(Quota)
			* Queries per day : 8,640,000
			* Max. QPS(Queries Per Second): 10
		* **하루 한도 사용량이 초과하더라도 Adavanced Tracker 동작에는 지장이 없음 (사용량 초과시 Tracker 모드와 동일하게 동작)**
	- 모드 설정은 다음과 같이 선언을 합니다.  (Recognizer, Tracker,  Advanced Tracker중 하나 선택)

		```java
		Plengi.getInstance(this).setMonitoringType(PlengiResponse.MonitoringType.STAY); // Recognizer mode
		Plengi.getInstance(this).setMonitoringType(PlengiResponse.MonitoringType.TRACKING); // Tracker mode
		Plengi.getInstance(this).setMonitoringType(PlengiResponse.MonitoringType.ADV_TRACKING); // Tracker mode
		```

* Advanced Tracker를 사용하기 위해서는 Android API Key 등록 및 설정이 필요 합니다.
	* [API Key 가져오기](https://developers.google.com/awareness/android-api/get-a-key#get_an_api_key_from_the_console_name) 페이지로  이동하여 아래의 그림 'GET A KEY' 버튼을 눌러 API KEY를 발급

	![get a key](https://storage.googleapis.com/loplat-storage/public/get_a_key.png)

	- 발급 받은 API Key를 아래의 샘플코드와 같이 YOUR_API_KEY란에 API key를 입력하여 AndroidManifest에 추가

		```xml
		<application>
			<!-- 중간 생략 -->
			<meta-data
				android:name="com.google.android.awareness.API_KEY"
				android:value="YOUR_API_KEY" />
			<!-- 이하 생략 -->
		</application>
		```

	* Awareness API 설명 및 API Key 등록과 관련 사항은 [Awareness API 설정](https://github.com/loplat/loplat-sdk-android/wiki/Advanced-Tracker-%EC%84%A4%EC%A0%95#advanced-tracker-setting) 페이지 참고부탁드립니다.
* **참고: 사용자 매장 방문 확인을 위해 기본으로 제공 되는 모드는 Recognizer 모드 입니다. Tracker/Advanced Tracker 모드를 사용하기 위해서는 협의가 필요 하오니 메일(yeddie@loplat.com)로 연락 바랍니다.** 

#### 2. WiFi 스캔 주기 설정
* 사용자의 매장/장소 방문 확인을 위한 WiFi Scan 주기를 설정합니다.
* WiFi scan 주기는 다음과 같이 설정합니다.
	```java
	Plengi.getInstance(this).setScanPeriod(3*60*1000, 6*60*1000);  // move: 3 mins, stay: 6 mins  
	```
		
	* Recognizer mode 일 경우  move, stay에 대해 주기를 설정합니다. 
		- move:  매장/장소를 인식하기 위한 기본 WFi scan 주기이며 default 값으로 3분이 설정되어 있습니다.  
			- 3분이하의 분으로 주기 설정시 default 값인 3분으로 설정이 됩니다.
		- stay: 매장/장소가 인식 된 후 WiFi scan 주기이며 default 값으로 2분이 설정되어 있습니다.  
			- 4분이하의 분으로 주기 설정시 default 값인 4분으로 설정이 됩니다.
	
	* Tracker mode 일 경우 분 단위로 설정이 가능하며 default 값으로 2분이 설정되어 있습니다.  
		```java
		Plengi.getInstance(this).setScanPeriodTracking(2*60*1000); // scanperiod: 2 mins 
		```
		- 1분이하의 분으로 주기 설정시 주기는 1분으로 설정이 됩니다. (최소 주기 값: 1분)
	* Advanced Tracker mode 일 경우 move, stay에 대해 주기를 설정합니다. 
		```java
		Plengi.getInstance(this).setScanPeriodAdvTracking(90*1000, 150*1000 ); // move: 1 min 30 sec, stay: 2 mins 30 sec 
		```
		- move:  매장/장소를 인식하기 위한 기본 WFi scan 주기이며 default 값으로 1분 30초 설정되어 있습니다.  
			- 1분 30초이하 주기 설정시 default 값이 1분 30초으로 설정이 됩니다.
		- stay: 매장/장소가 인식 된 후 WiFi scan 주기이며 default 값으로 2분 30초 설정되어 있습니다.  
			- 2분 30초이하 주기 설정시 default 값이 4분으로 설정이 됩니다.

#### 3. Gravity 연동하기
* Gravity 연동은 **SDK version 1.8.6**부터 연동이 가능합니다.
* **Gravity는 (필수항목) 위치 권한 허용, GPS on 상태에서 동작하오니 코드 작성시 유의하시기 바랍니다.**
*  Gravity 사용을 위해서 구글 ADID가 필수 항목이오니 [Library 적용하기](https://github.com/loplat/loplat-sdk-android#library-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0) 참고 하여 라이브러리 설정  부탁드립니다.
* **Gravity**를 통해 **푸쉬 메시지** (광고 및 알림 메시지)를 받기 위해서는 광고 알림 허용을 한 시점 아래와 같이 코드 작성이 필요 합니다.

	```java
	Plengi.getInstance(this).enableAdNetwork(true);            // 푸쉬 메시지 설정 on
	Plengi.getInstance(this).setAdNotiSmallIcon([samll icon id]);  // 푸쉬 메세지 small icon
	Plengi.getInstance(this).setAdNotiLargeIcon([large icon id]);  // 푸쉬 메세지 large icon
	 ```
        
#### 4. Start/Stop
- 사용자 장소/매장 방문 모니터링을 시작하거나 정지 할 수 있습니다.
- 설정된 주기마다 WiFi 신호를 스캔하여 사용자의 위치를 확인합니다.  
- **[주의] 모니터링 시작 전에 위치권한, GPS 상태, WiFi scan 가능 여부 등을 확인하는 과정이 필요 합니다. 3가지 조건을 확인하는 방법은 샘플 코드내에 구현된 checkWiFiScanCondition api(in MainActivity.java) 참고 부탁드립니다.**
- 사용자의 위치 정보는 PlengiEventListener로 전달됩니다.
-  모니터링 시작과 정지는 다음과 같이 선언합니다.  
	
	```java	
	Plengi.getInstance(this).start(); //Monitoring Start  
	Plengi.getInstance(this).stop(); //Monitoring Stop
	```

 - 모니터링 상태 확인은 Plengi.getEngineStatus를 통해서 확인 할 수 있습니다.
 	- 예시코드

		```java
		int engineStatus = Plengi.getInstance(this).getEngineStatus();
		if (engineStatus == PlaceEngine.EngineStatus.STARTED) {
			// Monitoring ON
		} else if (engineStatus == PlaceEngine.EngineStatus.STOPPED) {
			// Monitoring OFF
		}
		```
		
#### 5. 장소 인식 결과

* **참고**:
	1. SDK 1.7.5 이하 버전은 장소id는 loplatid(서버에 학습된 장소 id), placeid 둘 다 전달되며,  1.7.6 이상 버전 부터 장소 id는 loplatid로 통합되어 전달 됩니다.**
	2. SDK 1.8.6부터 장소 인식시 인식된 장소 결과에 따라 area(상권정보), complex(복합몰) 정보가 추가로 전달됩니다. 상권만 인식 된 경우에는 place 정보가 null로 넘어가니 코드 작성시 주의 부탁드립니다.
	3. SDK 1.8.6부터 lat_est, lng_est 항목은 삭제 되었습니다.

* 현재 위치가 인식 된 경우

	* 위치 정보 결과: **Place** (PlengiResponse.Place Class, response.place로 획득 가능)
		```java
		public long loplatid;        // 장소 id
		public String name;          // 장소 이름
		public String tags;          // 장소와 관련된 tag
		public int floor;            // 층 정보
		public String category;      // 장소 유형
		public String category_code; // 장소 유형 코드
		public double lat;           // 인식된 장소의 위도
		public double lng;	         // 인식된 장소의 경도
		public float accuracy;       // 정확도
		public float threshold;      // 한계치
		public double lat_est;       // 예측된 위치의 위도~~ v1.8.6에서 삭제
		public double lng_est;       // 예측된 위치의 경도, v1.8.6에서 삭제 
		public String client_code;   // 클라이언트 코드
		public String address;       // 장소 (구)주소
		public String address_road;  // 장소 신 주소
		public String post;           // 우편번호
		```
		* type: PlengiResponse.ResponseType.PLACE  
			- accuracy > threshold: 현재 위치 내에 있는 경우  
			- 그 외에 경우: 현재 위치 근처에 있는 경우 
	
	* 상권 정보 결과: **Area** (PlengiResponse.Area Class, response.area로 획득 가능)
		```java
		public int id;         // Area ID
		public String name;    // 상권 이름
		public String tag;     // 상권 위치 [도, 시 단위 ex) 서울, 경기도, 인천]
		public double lat;     // 위도 
		public double lng;     // 경도
		```
		* type: PlengiResponse.ResponseType.Area  
			- 장소 위치 요청한 장소가 상권 안일 경우 상권 정보가 인식 결과에 함께 같이 전달됩니다.
			-  위도 및 경도는 아래의 조건으로 결과가 전달됩니다.
				1. 장소 인식 결과값이 있다면 -> 인식된 장소 위도/ 경도
				2.  장소 인식 결과값이 없으면 -> device의 위도/경도
		
	* Complex 정보 결과: **Complex** (PlengiResponse.Complex Class, reponse.complex로 획득 가능)
		```java
		public int id;         // Complex ID
		public String name;    // 복합몰 이름
		public String branch_name;     // 복합몰 지점명
		public String category;     // 카테고리 
		public String category_code;     // 카테고리 코드
		```
		* type: PlengiResponse.ResponseType.Complex  
			* 인식된 장소가 복합몰 내인 경우 복합몰 정보도 함께 인식 결과에 포함되어 전달됩니다.
		
			        
* 현재위치 획득 실패시
	* type: PlengiResponse.ResponseType.PLACE
	* result: PlengiResponse.Result.ERROR_CLOUD_ACCESS
	* errorReason : Location Acquisition Fail  
	
* Client 인증 실패시
	* type: PlengiResponse.ResponseType.PLACE
	* result: PlengiResponse.Result.ERROR_CLOUD_ACCESS
	* errorReason : Not Allowed Client

### 6. API
#### 현재 위치 확인하기

* 현재 사용자가 위치한 장소/매장 정보를 loplat 서버를 통해 확인할 수 있습니다.  
* 현재 장소 정보를 서버에서 받아오고자 하는 경우 다음과 같은 선언을 합니다.

	```java
	Plengi.getInstance(this).refreshPlace();
	```
* WiFi AP들을 수집하여 loplat 서버에게 현재 사용자의 위치 정보를 요청합니다.
* loplat 서버는 최적의 위치정보를  PlengiEventListener로 전달합니다.  

* 자세한 사항은 API문서를 참조해주시기 바랍니다. [현재 위치 확인하기](https://github.com/loplat/loplat-sdk-android/wiki/API#%ED%98%84%EC%9E%AC%EC%9C%84%EC%B9%98-%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0)
	
#### 현재 사용자 상태 확인하기  (Stay or Move)
-  현재 사용자가 이동(Move) 중인지 매장/장소에 머무르고(Stay) 있는지 확인할 수 있습니다.
- 현재 사용자의 상태를 확인하기 위하여 다음과 같이 선언을 합니다. 
 
	 ```java
	Plengi.getInstance(this).getCurrentPlaceStatus();
	```
- 자세한 사항은 API문서를 참조해주시기 바랍니다. [현재 사용자 상태 확인하기](hhttps://github.com/loplat/loplat-sdk-android/wiki/API#%ED%98%84%EC%9E%AC-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%83%81%ED%83%9C-%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0)

#### 현재 장소 정보 가져오기
* 현재 사용자가 머무르고 있는 장소/매장 정보를 확인 할 수 있습니다.
* 현재 사용자가 위치한 장소/매장 정보를 확인하기 위하여 다음과 같이 선언을 합니다.
	```java
	Plengi.getInstance(this).getCurrentPlaceInfo();  
	```
* 매장/장소 정보는 PlengiResponse.Place로 전달됩니다. ([장소 인식 결과 참조](https://github.com/loplat/loplat-sdk-android/wiki/API#response))
* **참고사항**: 현재 사용자 상태가 STAY일 경우에만 정확한 장소/매장 정보를 획득 할 수 있습니다. 
* 자세한 사항은 API문서를 참조해주시기 바랍니다. [현재 장소 정보 가져오기](https://github.com/loplat/loplat-sdk-android/wiki/API#%ED%98%84%EC%9E%AC-%EC%9E%A5%EC%86%8C-%EC%A0%95%EB%B3%B4-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0)

## History
* 2018.04.22
	* loplat SDK version 1.8.9.4 release
		* 광고 알림 시 모바일 기기 화면이 켜지도록 수정
		* 광고 알림 시 sound profile대로 동작하도록 수정
		* 광고 설정시 small, large icon 구분하여 입력 하도록 수정
		* 일부 기능 개선

* 20.18.04.18
	* loplat SDK version 1.8.9.3 release
		* 인식 성능 개선 및 버그 수정

* 2018.04.10
	* loplat SDK version 1.8.9.2 release
		* 인식 성능 개선 및 버그 수정

* 2018.04.06
	* loplat SDK version 1.8.9.1 release
		* 허용되지 않는 계정인 경우 SDK 동작이 정지하도록 조치

* 2018.03.15
	* loplat SDK version 1.8.9 release
		* android 8.1(Oreo) 백그라운드 제약 사항 추가 대응
		* 일부 기능 및 인식 성능 개선, 버그 수정

* 2018.02.12
	* loplat SDK version 1.8.8 release
		* android 8.0(Oreo) 백그라운드 제약 사항 대응
	
* 2018.01.29
	* loplat SDK version 1.8.7 release
		* 인식 성능 개선 및 버그 수정

* 2018.01.22
  - loplat SDK version 1.8.6 release
	- Area, Complex 정보 추가
	- 인식 성능 개선 및 버그 수정

* 2018.01.17
   - loplat SDK version 1.8.5 release
	- Gravity 제공을 위한 기능 개선
	- Advanced Tracker 추가

* 2017.12.27
   - loplat SDK version 1.8.4 release
	- Gravity 연동
	
* 2017.11.23
   - loplat SDK version 1.8.3 release
	- 인식 성능 개선 및 버그 수정
	
* 2017.11.05
   - loplat SDK version 1.8.2 release 
	- 인식 성능 개선 및 버그 수정
	
* 2017.08.07
    - loplat SDK version 1.8.1 release
        - Nearby event, Enter Event 분리
	- EnterType class deprecated

* 2017.07.4
    - loplat SDK version 1.8.0 release
        - 업데이트 내용 : 인식 성능 개선 

* 2017.06.25
    - loplat SDK version 1.7.10 release
	    - 업데이트 내용
		    - Retrofit 라이브러리 적용
		    - 인식 성능 개선

* 2017.06.19
    - loplat SDK version 1.7.9 release
        - 업데이트 내용
            1. SDK 동작 상태를 확인 할 수 있는 기능 추가
* 2017.4.22
    - loplat SDK version 1.7.8 release
        - 업데이트 내용
            1. WiFi SSID 확인 중 발생하는 에러 보완 처리

* 2017.3.23
    - loplat SDK version 1.7.7 release
        - 업데이트 내용: PlengiListener 동작 보완

* 2017.3.7
	- loplat SDK version 1.7.6 release
		- 업데이트 내용
			1. 위치인식된 장소의 id를 loplatid 하나로 통합 (placeid는 더이상 전달되지 않음)
			2. 'unknown place'(학습되지 않은 장소)에 대한 enter/leave event 발생 중단
			3. PlengiResponse.EnterType 추가
            4. Plengi.getInstance(Context context).getVisitList(), Plengi.getInstance(Context context).getPlaceList() 삭제 
* 2016.12.20
    - loplat SDK version 1.7.5 release
        - 업데이트 내용
            1. Location Provider 획득 실패에 대한 예외 처리
            2. DB access error 보완
            3. LocationMonitorService 동작 확인 기능 추가
        
* 2016.12.13
    - loplat SDK version 1.7.4 release
        - 업데이트 내용
            1. DB access error 보완
            2. 위치 권한 미설정으로 WiFi Scan 결과값 획득 실패에 대한 보완 처리
            3. 위치 권한 확인 중 PackageManager가 죽는 현상에 따른 에러 보완 처리

* 2016.11.30
    - SDK version name 변경: 1.71 -> 1.7.1, 1.72 -> 1.7.2
    - lolat SDK version 1.7.3 release
        - 업데이트 내용
            1. wifi scan 요청 후, OS내에서 발생하는 에러에 대해 보완 처리

* 2016.11.17
    - loplat SDK version 1.72 release
        - 업데이트 내용: 일부 모델 wifi state access 에러에 대한 보완 처리  

* 2016.11.02
	- loplat SDK version 1.71 release
		- 업데이트 내용: Tracker Mode 장소 인식 개선

* 2016.10.17
    - 방문 매장/장소 기록 확인하기 (History of Places) function 삭제
    - **주의**: 현재 Plengi.getInstance(Context context).getVisitList()은 deprecated 되었으니, 이점 유의 해주시길 바랍니다.

* 2016.10.11
	* loplat SDK version 1.7 release
		* 업데이트 내용
			1. Init시 uniqueUserId 업데이트 관련 버그 개선
			2. 일부매장에서 현재요청시 발생하는 에러에 대해 보완 처리 
	
* 2016.08.09
	* loplat SDK version 1.6 release
		* 업데이트 내용
		    1. Init시 uniqueUserId 수정이 가능하도록 변경
		    2. BOOT_COMPLETED시 SDK 자동 재시작 설정
		    3. 편의점 매장 인식 속도 개선

* 2016.05.19
	* loplat SDK version 1.5 release 
		- 업데이트 내용
			1. 일부 모델 간헐적 db access 에러에 대해 보완 처리
			2. Tracking Mode, Tracking Event 추가
			3. 위치 정보 결과에 loplatid 추가
			
* 2016.04.22
	* loplat SDK version 1.4 release
		- 업데이트 내용: library 포함하여 build 시에 proguard 관련 오류 제거


* 2016.02.12 
	* loplat SDK version 1.3 release
		* 업데이트 내용
			1. 장소 정보에 client_code 추가
			2. 동일 client_code를 가지는 장소 내에서의 이동 시에 중복해서 ENTER Event를 발생하지 않도록 변경
		- **주의**: 이전 library와 db 호환이 되지 않아, 이전버전으로 만든 앱은 꼭 삭제 후 새 버전 설치
	- 장소학습기 (loplat cook) 릴리즈
	

* 2016.01.27 - initial release
