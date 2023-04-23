---
layout: "post"
title: "[TIL] md파일 작성법"
subtitle: "마크다운 이용 가이드"
author: "raindragon"
banner:
  image: "/assets/images/base/banner.jpg"
  opacity: 0.618
  height: "38vh"
  min_height: "38vh"
  heading_style: "font-size: 2.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
lang: ko
tags: [Markdown]
---

<!-- Heading -->
<!-- 헤딩 스타일  -->
<!-- 
    heading 1의 경우 밑줄이 쳐진다
 -->

![](https://youtu.be/kMEb_BzyUqk)

우연히 유튜브에서 [드림코딩엘리님](https://youtu.be/kMEb_BzyUqk)의 Markdown 작성법을 보고 블로그를 시작하게 하면서,   
해당 영상의 내용을 정리해 보았다.

<br/><br/>

# Heading 1 - `# Heading 1`
## Heading 2 - `## Heading 2`
### Heading 3 - `### Heading 3`
#### Heading 4 - `#### Heading 4`
##### Heading 5 - `##### Heading 5`
###### Heading 6 - `###### Heading 6`

<!-- 일반 텍스트 -->
일반 텍스트 
<br/><br/>

<!-- Line 추가 -->

라인 추가 - `___`

___

<br/><br/>

텍스트
=


 `**볼드체**` - **볼드체**  

 `*이태릭*` -  *이태릭*  
 
 `~~텍스트 가운데 줄~~` - ~~텍스트 가운데 줄~~  

 <br/><br/>

인용구 (Quote)
===

 > 인용구 넣기 - `>`  


<br/><br/>

리스트 만들기
===

Bullet list - 앞에 *,-,1달기

 리스트
 * 목록1
 * 목록2
- 목록3
- 목록4
1. 목록5
2. 목록6


<br/>

링크 만들기
===

`[단어](링크)`
Click [here](https://github.com/raindragonn)

`![이미지 설명](이미지 링크)` 이미지 추가

![이미지설명](https://images.unsplash.com/photo-1498050108023-c5249f4df085?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1652&q=80)



테이블 만들기
===

`|Header|Description|`<br/>
`|:--:|:--:|` 콜론의 위치로 정렬 가능 양쪽 콜론 - 가운데 정렬<br/>
`|cell1|cell2|`<br/>
`|cell1|cell2|`<br/>
`|cell1|cell2|`<br/>


|Header|Description|
|:--:|:--:|
|cell1|cell2|
|cell1|cell2|
|cell1|cell2|

<br/><br/>

코드 작성
===

![코드][code]


```kotlin
fun main(){
    private val ad : Int = 1
}
```


공백 4개 일때도 가능하다

    fun main(){
        private val ad : Int = 1
    }
    
end code block.   
<br/>

[그외 잘 정리된 블로그][총정리]

   <br/><br/><br/>

참고

___

1. [마크다운 작성법][markdown]
2. [드림코딩 엘리][엘리]



[code]: /assets/images/post/2021-01-12.png "코틀린 사용"
[markdown]: https://gist.github.com/raindragonn/7e64da15b9365a5e8d6b53577b71ef77 "마크다운 작성법"
[엘리]: https://youtu.be/kMEb_BzyUqk
[총정리]: https://heropy.blog/2017/09/30/markdown/
