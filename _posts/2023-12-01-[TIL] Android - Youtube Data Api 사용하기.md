---
layout: "post"
title: "[TIL] Android - Youtube Data API를 사용해서 동영상 검색하기(OAuth, Api key)"
subtitle: "Youtube Data Api"
date: 2023-12-08
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
  - Android
  - Youtube
---

## [Android] Youtube Data API - 동영상 검색(Oauth, Api key)

과거 유튜브 알람앱을 만들때에는 유튜브 동영상의 공유를 통해 아이디를 받아와서 사용하도록 했었지만 

새롭게 출시할 앱은 앱 내에서 검색 기능도 추가할 겸 Youtube Data Api를 사용하는 방법을 알아보았다.

해당 기능은 물론 `Retrofit`과 같은 HTTP 통신을 통해서도 검색이 가능하지만 [Quick Starter](https://developers.google.com/youtube/v3/quickstart/android?hl=ko) 를 기반으로 작성했다.

## Youtube Data API

- Youtube 웹사이트에서 일반적으로 실행되는 기능을 웹 혹은 앱에서 사용할 수 있다.
- 오늘 사용할 것은 그중에서도 [컨텐츠 검색](https://developers.google.com/youtube/v3/docs/search/list?hl=ko)이다.

## 선행 작업

### 1.  Youtube Data Api 사용 설정

- [[링크]](https://console.developers.google.com/start/api?id=youtube&hl=ko)를 통해 GCP console로 이동해서 프로젝트를 확인하고 사용 설정을 해준다.
    
    ![Untitled](/assets/images/post/2023-12-08/1.png)
    
### 2 - 1 - 1.[ OAUTH 동의를 이용하는 경우 ] OAUTH 동의 화면 등록

- Oauth 동의 화면을 등록해 준다. (해당 예제는 기능 테스트만을 위한 것입니다.)
    
    ![Untitled](/assets/images/post/2023-12-08/2.png)
    
    ![Untitled](/assets/images/post/2023-12-08/3.png)
    
    ![Untitled](/assets/images/post/2023-12-08/4.png)
    

### 2 - 1 - 2.  사용자 인증 정보 추가

- 페이지 좌측 [ 사용 설정된 API 및 서비스 ] → 목록 중  [ Youtube Data Api V3 ]
- 사용자 인증정보 탭에서 [ 사용자 인증 정보 만들기 ] → [ OAuth 클라이언트 ID ] 클릭
    
    ![Untitled](/assets/images/post/2023-12-08/5.png)
    
- 이후 패키지 이름 및 SHA1을 등록해준다. [[ 참고 ]](https://developers.google.com/youtube/v3/quickstart/android?hl=ko#step_1_acquire_a_sha1_fingerprint)
    
    ![Untitled](/assets/images/post/2023-12-08/6.png)
    
- 테스트 앱게시 클릭
    
    ![Untitled](/assets/images/post/2023-12-08/7.png)
    

### 2 - 2.[ API KEY를 이용하는 경우 ]

- 페이지 좌측 [ 사용 설정된 API 및 서비스 ] → 목록 중  [ Youtube Data Api V3 ]
- 사용자 인증정보 탭에서 [ 사용자 인증 정보 만들기 ] → [ API 키 ] 클릭
    
    ![Untitled](/assets/images/post/2023-12-08/8.png)
    
- Android 제한 사항에 패키지, SHA1 추가 후 API KEY 사용
    
    ![Untitled](/assets/images/post/2023-12-08/9.png)
    

## 1. 라이브러리 추가

- 각 라이브러리 별 최신 버전은 https://mvnrepository.com/ 참조

```kotlin
dependencies {
	...
	implementation("com.google.android.gms:play-services-auth:20.7.0")
	implementation("com.google.apis:google-api-services-youtube:v3-rev222-1.25.0")
	implementation("com.google.api-client:google-api-client-android:2.2.0")
	...
}
```

- `2 files found with path 'META-INF/DEPENDENCIES'.` 에러 발생 시 아래 구문 추가
    
    ```kotlin
    android {
    	...
    	packaging {
    		resources.excludes.add("META-INF/DEPENDENCIES")
    	}
    	...
    }
    ```
    

## 2. 권한 추가

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
```

## 3. 샘플 코드 1 - OAUTH 인증 사용 예시

```kotlin

class MainActivity : AppCompatActivity() {
    companion object {
        const val PREF_NAME = "PREF_NAME"
        const val KEY_SELECTED_NAME = "KEY_SELECTED_NAME"
    }

    private lateinit var _etSearch: EditText
    private lateinit var _btnSearch: Button
    private lateinit var _tvResult: TextView

    private val searchText
        get() = _etSearch.text.toString()

    private val _credential: GoogleAccountCredential by lazy {
        GoogleAccountCredential.usingOAuth2(
            applicationContext,
            listOf(YouTubeScopes.YOUTUBE_READONLY),
        ).setBackOff(ExponentialBackOff())
    }

    private val _credentialLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult(), ::credentialCallBack
    )

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        _etSearch = findViewById(R.id.et_search)
        _btnSearch = findViewById(R.id.btn_search)
        _tvResult = findViewById(R.id.tv_result)

        _btnSearch.setOnClickListener {
            search()
        }
    }

    private fun search() {
        if (checkSignIn()) {
            lifecycleScope.launch {
                val result = youtubeSearch(searchText).joinToString("\n") { it.snippet.title }
                _tvResult.text = result
            }
        }
    }

    private fun checkSignIn(): Boolean {
        val lastSelectedAccountName = getSharedPreferences(PREF_NAME, MODE_PRIVATE)
            .getString(KEY_SELECTED_NAME, null)
        return if (lastSelectedAccountName.isNullOrBlank()) {
            val signInIntent = _credential.newChooseAccountIntent()
            _credentialLauncher.launch(signInIntent)
            false
        } else {
            _credential.selectedAccountName = lastSelectedAccountName
            true
        }
    }

    private fun credentialCallBack(result: ActivityResult) {
        val accountName = result.data?.getStringExtra(AccountManager.KEY_ACCOUNT_NAME) ?: return

        getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE).edit {
            putString(KEY_SELECTED_NAME, accountName)
        }
        _credential.selectedAccountName = accountName

        Toast.makeText(this, "검색 가능", Toast.LENGTH_SHORT).show()
    }

    private suspend fun youtubeSearch(query: String): List<SearchResult> {
        val transport = NetHttpTransport()
        val jsonFactory: JsonFactory = JacksonFactory.getDefaultInstance()
        val youtube = YouTube.Builder(
            transport, jsonFactory, _credential
        ).setApplicationName(getString(R.string.app_name))
            .build()

        return withContext(Dispatchers.IO) {
            runCatching {
                // 매개변수 참고 [https://developers.google.com/youtube/v3/docs/search/list?hl=ko#parameters]
                val parts = listOf("id", "snippet").joinToString()
                val searchList =
                    youtube.search().list(parts).apply {
                        q = query
                        type = "video"
                        // 음악 카테고리 참고 [https://gist.github.com/dgp/1b24bf2961521bd75d6c]
                        videoCategoryId = "10"
                    }
                val response = searchList?.execute()
                response?.items ?: listOf()
            }.recover { throwable ->
                if (throwable is UserRecoverableAuthIOException) {
                    launch(Dispatchers.Main) {
                        Toast.makeText(this@MainActivity, "인증이 필요합니다.", Toast.LENGTH_SHORT).show()
                        // 엑세스 권한이 없는 경우 권한 요청을 위한 Intent
                        val accessRequestIntent = throwable.intent
                        _credentialLauncher.launch(accessRequestIntent)
                    }
                }
                listOf()
            }.getOrNull() ?: listOf()
        }
    }

}
```

- `GoogleAccountCrecdential` 을 통해 Google 계정에 대한 승인 및 계정 선택을 관리한다.
- `GoogleAccountCrecdential.newChooseAccountIntent()` 메서드를 통해 계정 선택이 가능하다.
    - 계정 선택만으로 Youtube 검색은 불가능하다.
    - 선택한 계정이 액세스 권한이 있어야한다.
- 검색에 대한 예외로 액세스 권한이 없는경우 `UserRecoverableAuthIOException` 예외가 발생하며, 해당 예외에서 액세스 권한 요청이 가능한 `intent`를 가져올 수 있다.

## 3. 샘플코드 2 - API KEY 사용하기

```kotlin
class MainActivity : AppCompatActivity() {
    companion object {
        const val YOUTUBE_DATA_API_KEY = "Youtube Data API 키"
    }

    private lateinit var _etSearch: EditText
    private lateinit var _btnSearch: Button
    private lateinit var _tvResult: TextView

    private val searchText
        get() = _etSearch.text.toString()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        _etSearch = findViewById(R.id.et_search)
        _btnSearch = findViewById(R.id.btn_search)
        _tvResult = findViewById(R.id.tv_result)

        _btnSearch.setOnClickListener {
            search()
        }
    }

    private fun search() {
        lifecycleScope.launch {
            val result = youtubeSearch(searchText).joinToString("\n") { it.snippet.title }
            _tvResult.text = result
        }
    }

    private suspend fun youtubeSearch(query: String): List<SearchResult> {
        val transport = NetHttpTransport()
        val jsonFactory: JsonFactory = JacksonFactory.getDefaultInstance()
        val youtube = YouTube.Builder(
            transport, jsonFactory,
        ) {}.setApplicationName(getString(R.string.app_name))
            .build()

        return withContext(Dispatchers.IO) {
            runCatching {
                // 매개변수 참고 [https://developers.google.com/youtube/v3/docs/search/list?hl=ko#parameters]
                val parts = listOf("id", "snippet").joinToString()
                val searchList =
                    youtube.search().list(parts).apply {
                        key = YOUTUBE_DATA_API_KEY
                        q = query
                        type = "video"
                        // 음악 카테고리 참고 [https://gist.github.com/dgp/1b24bf2961521bd75d6c]
                        videoCategoryId = "10"
                    }
                val response = searchList?.execute()
                response?.items ?: listOf()
            }.recover {
                it.printStackTrace()
                listOf()
            }.getOrElse { listOf() }
        }
    }

}
```

- 유저에게 액세스 권한을 물을 필요 없이 `API KEY`를 이용하면 된다.

## 깃헙에서 확인하기.

- 해당 예제에 대한 소스는 [[ 링크 ]](https://github.com/raindragonn/YoutubeDataApiTest) 를 통해 확인이 가능합니다.
- **OAuth, API Key 사용은 각각 브랜치를 확인해주세요.**
    
    

## 사용 후기

- OAUTH를 통한다면 권한을 통해 검색뿐아니라 보다 다양한 작업이 가능할 것으로 보인다.
- 각 API 별 [할당량](https://developers.google.com/youtube/v3/determine_quota_cost?hl=ko) 이 있는데 그 중 검색은 하루에 100번 검색이면 할당량을 다쓰는 만큼 굉장히 크다.
- 처음 API KEY로만 사용할 때와 OATUH를 통한 검색의 할당량의 차이가 있을까 싶었지만 둘이 같은것으로 보여진다.

### 참고 자료

---

[Youtube Data Api](https://developers.google.com/youtube/v3/quickstart/android?hl=ko)