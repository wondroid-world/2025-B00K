# RecyclerView로 리스트 보여주기

## RecyclerView 추가하기

```kotlin
// build.gradle.kts
dependencies {
    implementation(libs.androidx.recyclerview)
}

// libs.versions.toml
[libraries]
androidx-recyclerview = { group = "androidx.recyclerview", name = "recyclerview", version = "1.3.2" }
```

→ 이 방법은 버전을 통합해서 관리하는 방법임

```xml
<android.recyclerview.widget.RecyclerView
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:id="+@id/crime_recycler_view"
	android:layout_width= "match_parent"
	android:layout_height="match_parent" />
```

```kotlin
crimeRecyclerView = view.findViewById(R.id.crime_recycler_view) as RecyclerView
crimeRecyclerView.layoutManager = LinearLayoutManager(context)
```

RecyclerView에 `LayoutManager`를 설정하지 않으면 작동이 중단되는 이유는 **RecyclerView는 LayoutManager가 있어야 아이템 뷰를 어떻게 배치하고 스크롤할지를 결정할 수 있기 때문**입니다.

### 🔧 좀 더 구체적으로 설명하면:

- `RecyclerView`는 **자체적으로 아이템 배치를 담당하지 않음**.
- `LayoutManager`는 아이템들을 수직/수평으로 배치하거나, 격자로 배치하거나, 또는 커스텀한 방식으로 배치하는 **전략 객체**.
- 따라서 `LayoutManager`가 없으면 `RecyclerView`는 "어떻게 보여줘야 할지"를 몰라서 아예 동작을 멈춥니다.
- LinearLayoutManager → 수직
- GridLayoutManager → 격자 형태

---

## 항목 뷰 레이아웃 생성하기

- RecyclerView는 ViewGroup의 서브 클래스
- 항목 뷰(item view)라고 하는 자식 view 객체들의 리스트를 보여줌

---

## ViewHolder 구현하기

- recyclerView는 항목 View가 ViewHolder 인스턴스에 포함되어 있다.
    
    → RecyclerView는 아이템 뷰를 ViewHolder 안에 넣어서 재사용하고 관리한다.
    
- ViewHolder는 항목 View의 참조를 갖는다.
    
    → 아이템을 ViewHolder에 넣어서 리스트로 관리
    

```kotlin
class MovieListAdapter(
    private val value: List<Movie>,
    private val navigateToBook: (Movie) -> Unit,
    private val navigateToAd: () -> Unit,
) : RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    override fun onCreateViewHolder(
        parent: ViewGroup,
        viewType: Int,
    ): RecyclerView.ViewHolder {
        return when (viewType) {
            VIEW_TYPE_MOVIE -> {
                val binding =
                    MovieItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
                MovieViewHolder(binding)
            }

            else -> {
                val binding =
                    AdItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
                AdViewHolder(binding)
            }
        }
   
   .
   .
   . 
    
        inner class MovieViewHolder(
        private val binding: MovieItemBinding,
    ) : RecyclerView.ViewHolder(binding.root) {
        fun bindMovie(movie: Movie) {
            binding.root.setOnSingleClickListener { navigateToBook(movie) }
            binding.movie = movie
            binding.movieBookBtn.setOnSingleClickListener { navigateToBook(movie) }
        }
    }
```

| 구성요소 | 역할 |
| --- | --- |
| RecyclerView | 뷰 직접 안 만듦. 대신 Adapter에 위임 |
| Adapter | View를 만들어서 ViewHolder로 감싸줌 |
| ViewHolder | 아이템 View의 참조를 저장하고 관리함 |

---

## 어댑터를 구현해 RecyclerView에 데이터 채우기

- recyclerView는 자신이 viewHolder를 생성하지 않는다. → Adapter에 요청

Adapter가 하는 일

- 새로운 viewHolder 인스턴스의 생성을 어댑터에 요청
- 지정된 위치의 데이터 항목에 viewHolder를 바인딩하도록 어댑터에 요청
- List를 RecyclerView는 모르며, Crime Adapter가 안다.

순서

- recyclerView는 crimeAdapter의 onCreateViewHolder 함수를 호출해 CrimeHolder 인스턴스를 생성
- CrimeAdapter가 생성해 RecyclerView에게 반환하는 CrimeHolder는 아직 데이터가 바인딩되지 않았다.
- CrimeHolder와 데이터 셋 내부의 Crime 객체의 위치를 인자로 전달

---

## 뷰의 재활용: RecylcerView

- recyclerView는 화면을 채우는 데 충분한 개수만 생성해, 화면이 스크롤되면서 항목 View가 화면을 벗어날 때 view를 버리지 않고 재활용

---

## 리스트 항목의 바인딩 개선하기

- 데이터 바인딩 작업을 수행하는 모든 코드는 CrimeHolder 내부에 두는 것이 좋다.
- 클릭 이벤트 리스너도 Holder에서 구현