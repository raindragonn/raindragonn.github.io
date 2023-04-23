---
layout: "post"
title: "[TIL] Android - Paging 3"
subtitle: "Paging 3"
date:       2021-07-30
author: "raindragon"
banner:
  image: "/assets/images/base/banner.jpg"
  opacity: 0.618
  height: "38vh"
  min_height: "38vh"
  heading_style: "font-size: 2.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
lang: ko
categories: [TIL - Android]
tags:
    - AAC
    - Paging3
---

최근에 사전과제를 진행하면서 처음으로 [페이징][페이징] 라이브러리를 사용하며 공부한 내용입니다.


### 페이징이란?
페이징은 대량의 데이터를 한 번에 불러오는 것이 아니라

필요한 만큼 `일부분을 나눠서 가져오는 것`을 의미합니다.

API에 따라 `limit(한 번에 보여줄 데이터의 수), offset(데이터의 인덱스)` 등으로

페이징 처리 되어 있습니다.

## Android JetPack Paging 3 사용 이유

1. 페이징 된 데이터의 메모리 캐싱으로 시스템 리소스를 효율적으로 사용할 수 있다.
2. 요청 중복 제거 기능 지원
3. RecyclerView 의 스크롤 할 때 자동으로 데이터로드를 지원
4. 새로 고침, 재시도, 오류 처리를 지원
 

## Paging3 아키텍처

페이징 라이브러리는 [안드로이드 권장 아키텍처][권장아키텍처]에 맞게 설계 되어있으며 총 3개의 Layer로 구성 됩니다.

1. Repository Layer
- `PagingSource` : 데이터 소스를 정의하고 데이터를 가져오는 방법을 정의
- `RemoteMediator` : 로컬 데이터베이스를 이용해 네트워크 데이터를 캐싱하기 위해 사용됩니다.(7월 29일 기준 실험용으로 나와있으며 향후 변경될 수 있습니다.)

2. ViewModel Layer
- `Pager` : PagingSource를 리턴타입으로 하는 람다와 PagingConfig를 생성자로 받으며 PagingData 를 반응형 스트림으로 생성할 수 있습니다.
- `PagingData` : 페이징된 데이터를 담아두는 컨테이너 입니다. Paging Source에서 load한 결과를 저장하며 UI Layer의 PagingDataAdapter로 넘겨 줍니다.

3. UI Layer
- `PagingDataAdapter` : Paging3 에서 지원하는 RecyclerView 의 어댑터이며, PagingData 를 받아서 처리해줍니다.(ListAdapter 와 같이 DiffUtil 을 통해 데이터를 새로고침 해 줍니다.)


## 사용법

본 예제는 [Itunes Search Api][Itunes]를 이용하겠습니다.

###  1. 의존성 추가하기

```groovy
dependencies {
    implementation "androidx.paging:paging-runtime-ktx:3.0.1"
}
```

### 2. PagingSource 작성

```kotlin
/**
 * 데이터 소스 정의 및 데이터를 가져오는 방법 정의
 * 키의 유형은 offset 이용 하므로 Int
 * 데이터 유형은 TrackItem
 * 데이터를 가져오는 위치 = RemoteDataSource
 */
class ItunesPagingSource(
    private val remoteDataSource: RemoteDataSource
) : PagingSource<Int, TrackItem>() {
    /**
     * 사용자가 스크롤할 때 데이터를 비동기식으로 가져오기 위한 Load 함수 구현
     * LoadParams 객체에는 로드할 페이지의 키 와 로드 크기 정보가 저장됩니다.
     * 초기에는 key 가 null 이며, loadSize 는 가져온 데이터의 크기를 뜻합니다.
     */
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, TrackItem> {
        val position = params.key ?: 0

        return try {
            val response = remoteDataSource.getTrackItemList(
                limit = params.loadSize,
                offset = position
            )

            val nextKey = if (response.isEmpty()) {
                null
            } else {
                position + params.loadSize
            }

            LoadResult.Page(
                data = response,
                prevKey = if (position == 0) null else position - 1,
                nextKey = nextKey
            )
        } catch (exception: IOException) {
            LoadResult.Error(exception)
        } catch (exception: HttpException) {
            LoadResult.Error(exception)
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    /**
     * 새로고침 시 사용됩니다.
     * 가장 최근 인덱스를 anchorPosition 을 이용해 주변 데이터의 로드를 다시 시작
     */
    override fun getRefreshKey(state: PagingState<Int, TrackItem>): Int? {
        // We need to get the previous key (or next key if previous is null) of the page
        // that was closest to the most recently accessed index.
        // Anchor position is the most recently accessed index
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }
}
```


`abstract class PagingSource<Key : Any, Value : Any>`를 상속받아서 ItunesPagingSource를 만들어줍니다.

제네릭타입 2가지에서 페이지 정보인 key는 Int로, 사용하는 데이터인 value 즉 데이터 유형은 TrackItem으로 하였습니다.

`abstract fun인 load() 와 getRefreshKey()`를 구현 해야 합니다.

**`load()`**는 인자로 넘어오는 `LoadParams`객체를 바탕으로 페이지의 데이터를 반환 합니다.

`LoadParams`는 key값과 loadSize를 가지고 있습니다.

제가 이번 예제에 사용하는 Itunes Search Api는 데이터를 가져오는 크기인 limit와 데이터의 시작위치 offset을 필요로 하기에 `LoadParams`의 key와 loadSize를 이용해 데이터를 가져오도록 하였습니다.

리턴 타입은 `LoadResult`로, Page, Errer 두 가지가 있으며,정상적인 경우 `LoadResult.Page(data, prevKey, nextKey)` 를 통해 데이터, 이전 키, 다음 키 를 넘겨 주고 다음 데이터를 더 이상 호출하지 않을 때는 nextKey 값을 null로 주면 됩니다.

에러나 데이터에 문제가 있을 경우 `LoadResult.Error(throwable)` 를 넘겨 주면 됩니다.

**`getRefreshKey()`**는 새로고침할때 다시 시작할 key를 반환 합니다.

인자로 넘어오는 `PagingState` 객체는 인자로 받아서 로드된 페이지 및 마지막으로 액세스 한 위치 등의 페이징 시스템의 스냅샷 상태를 가지고 있습니다.


### 3. Repository 구현

```kotlin
class TrackRepositoryImpl(
    private val remoteDataSource: RemoteDataSource
) : TrackRepository {

    override fun getTrackList(): LiveData<PagingData<TrackItem>> {
        return Pager(
            config = PagingConfig(
                pageSize = SEARCH_LIMIT_SIZE,
                enablePlaceholders = false
            ),
            pagingSourceFactory = {
                ItunesPagingSource(remoteDataSource)
            }
        ).liveData
    }
}
```

Repository에서 `Pager`의 생성자로 `PagingConfig`와 `PagingSource를 리턴하는 람다`를넘기고 `LiveData<PagingData<TrackItem>>`로 반환해줍니다.

`PagingConfig`에서 pageSize가 이전에 구현한 PagingSource의 load()에 인자인 `LoadParams의 loadSize`가 됩니다. (첫 로드 시 *3 한 값이 넘어갑니다 [링크](https://developer.android.com/codelabs/android-paging?hl=ko#4) 맨 아래 참고)


### 4. ViewModel 구현

```kotlin
class MainViewModel(
    private val repository: TrackRepository
) : ViewModel() {

    private val _trackList: MutableLiveData<PagingData<TrackItem>> =
        repository.getTrackList().cachedIn(viewModelScope) as MutableLiveData<PagingData<TrackItem>>

    val trackList: LiveData<PagingData<TrackItem>>
        get() = _trackList
}
```
Repository를 통해 `PagingData`를 가져옵니다. cachedIn(viewModelScope)를 사용하여 캐싱을 해줄 수 있습니다.


### 5. Paging Adapter, Activity 구현

```kotlin
class TrackPagingAdapter : PagingDataAdapter<TrackItem, TrackViewHolder>(differ) {
    companion object {
        private val differ = object : DiffUtil.ItemCallback<TrackItem>() {
            override fun areItemsTheSame(oldItem: TrackItem, newItem: TrackItem): Boolean {
                return oldItem.trackId == newItem.trackId
            }

            override fun areContentsTheSame(oldItem: TrackItem, newItem: TrackItem): Boolean {
                return oldItem.trackName == newItem.trackName &&
                        oldItem.artistName == newItem.artistName
            }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TrackViewHolder {
        return TrackViewHolder.create(parent)
    }

    override fun onBindViewHolder(holderTrack: TrackViewHolder, position: Int) {
        getItem(position)?.let { trackItem ->
            holderTrack.bind(trackItem)
        }
    }
}
```

```kotlin
class TrackViewHolder(
    private val binding: ItemTrackBinding
) : RecyclerView.ViewHolder(binding.root) {
    companion object {
        fun create(
            parent: ViewGroup
        ): TrackViewHolder {
            val binding = ItemTrackBinding.inflate(
                LayoutInflater.from(parent.context),
                parent,
                false
            )

            return TrackViewHolder(binding)
        }
    }

    fun bind(track: TrackItem) {
        binding.trackItem = track
    }
}
```

Paging Adapter의 경우 ListAdapter처럼 구현해 주면 됩니다.

```kotlin
class MainActivity : AppCompatActivity() {
    private val binding: ActivityMainBinding by lazy {
        DataBindingUtil.setContentView(this, R.layout.activity_main)
    }
    private val viewModel: MainViewModel by viewModel()
    private val adapter = TrackPagingAdapter()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding.vm = viewModel

        initRecyclerView()
        onObserve()
    }

    private fun initRecyclerView() {
        binding.rv.adapter = adapter
    }

    private fun onObserve() {
        viewModel.trackList.observe(this) {
            adapter.submitData(lifecycle, it)
        }
    }
}
```

adapter의 submitData를 통해 pagingData를 넘겨주면 됩니다.


**예제 코드는 [여기서][git] 확인할 수 있습니다.**

---

참고자료

[코드랩](https://developer.android.com/codelabs/android-paging?hl=ko#0)

[꾸준하게님 블로그](https://leveloper.tistory.com/202)


[페이징]:https://developer.android.com/topic/libraries/architecture/paging/v3-overview?hl=ko
[권장아키텍처]:https://developer.android.com/jetpack/guide#recommended-app-arch
[Itunes]:https://affiliate.itunes.apple.com/resources/documentation/itunes-store-web-service-search-api/
[git]:https://github.com/raindragonn/pagingStudy