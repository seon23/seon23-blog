---
title: "Python 페이지 스크래핑 후 특정 문자열 제거"
date: "2022-08-27"
slug: "remove-string-python"
hero_image: "./python-strip"
# hero_image_alt: ""
# hero_image_credit_text: ""
# hero_image_credit_link: ""
---

...

스파르타 코딩클럽 웹개발 종합반 2주차 숙제를 하던 중 문제가 발생했다.

## 작업 설명 및 예상 결과

1. 음원사이트의 Top50 목록을 스크래핑한다. 
2. 스크래핑할 사이트
    - [https://www.genie.co.kr/chart/top200?ditc=M&rtm=N&ymd=20210701](https://www.genie.co.kr/chart/top200?ditc=M&rtm=N&ymd=20210701)
3. 예상 결과
   <p align="center">
    <img src = "../../src/images/scrape-expected-result" width="90%">
   </p> 

<br>

## 오류 설명 및 재현

### 1. 기존 코드와 실행 결과

```python
import requests
from bs4 import BeautifulSoup

headers = {'User-Agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'}
data = requests.get('https://www.genie.co.kr/chart/top200?ditc=M&rtm=N&ymd=20210701',headers=headers)
soup = BeautifulSoup(data.text, 'html.parser')

songs = soup.select('#body-content > div.newest-list > div > table > tbody > tr')

for song in songs:
    rank = song.select_one('td.number').text[0:2].strip()
    title = song.select_one('td.info > a.title.ellipsis').text.strip()
    artist = song.select_one('td.info > a.artist.ellipsis').text.strip()

    print(rank, title, artist)
```

![https://velog.velcdn.com/images/govkfl100/post/9fc248f1-6e5a-4b94-9468-a66dd019c6bd/image.png](https://velog.velcdn.com/images/govkfl100/post/9fc248f1-6e5a-4b94-9468-a66dd019c6bd/image.png)

15위 곡 제목 옆에 '19금' 문자와 긴 공백이 출력되었다. title 변수 할당 과정을 수정해야한다.

<br>

### 2. 원인과 해결 과정

![https://velog.velcdn.com/images/govkfl100/post/89b61ca1-b128-4840-a947-32efa92134b9/image.png](https://velog.velcdn.com/images/govkfl100/post/89b61ca1-b128-4840-a947-32efa92134b9/image.png)

- 15위 Peaches에 달려있는 19마크가 문제이므로, 이 부분을 구성하는 코드를 직접 살펴보았다. 해당 코드는 다음과 같다.
    
    ```html
    <a href="#" class="title ellipsis" title="재생" onclick="fnPlaySong('92682943','1');return false;">
    	<span class="icon icon-19">
        19	<!--a.title.ellipsis.text가 가리키는 부분(1)-->
        <span class="hide">
          금	<!--a.title.ellipsis.text가 가리키는 부분(2)-->
        </span>
    	</span>
    	<!--a.title.ellipsis.text가 가리키는 부분(3)-->
      Peaches (Feat. Daniel Caesar &amp; Giveon)
    </a>
    ```
    
- <a></a> 안에 보이는 ‘19, 금, Peaches (Feat. Daniel Caesar &amp; Giveon)’이 `a.title.ellipsis.text`에 해당한다. 다른 곡들과 다르게 text가 가리키는 부분에 `19`, `금`이 포함되어 있기 때문에, text를 호출하면 이 문자가 따라올 수 밖에 없다.

따라서 불필요한 문자를 발견하면 삭제하는 코드를 for문 내에 삽입해 보았다.

> 시도 1) strip('char') 으로 특정 문자 제거
> 
> 
> ```python
> # 기존 코드 for문에 추가할 내용.
> if '19금' in title:
> 	title = title.strip('19금 ')
> print(title)
> ```
> 
> - '19금'뒤에 포함된 긴 공백까지 삭제하고자 '19금 '을 인자로 주었으나, 여전히 15위 곡 부분에서 공백이 사라지지 않았다.

> 시도 2) strip 연달아 적용해서 문자열, 공백 차례로 제거
> 
> 
> ```python
> if '19금' in title:
> 	title = title.strip('19금').strip()
> print(title)
> ```
> 
> ```python
> import requests
> from bs4 import BeautifulSoup
> 
> headers = {'User-Agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'}
> data = requests.get('https://www.genie.co.kr/chart/top200?ditc=M&rtm=N&ymd=20210701',headers=headers)
> soup = BeautifulSoup(data.text, 'html.parser')
> 
> songs = soup.select('#body-content > div.newest-list > div > table > tbody > tr')
> 
> for song in songs:
>     rank = song.select_one('td.number').text[0:2].strip()
>     title = song.select_one('td.info > a.title.ellipsis').text.strip()
>     if '19금' in title:
>         title = title.strip('19금').strip()
> 
>     artist = song.select_one('td.info > a.artist.ellipsis').text
>     print(rank, title, artist)
> ```
> 
> - 15위 곡이 정상적으로 출력된다.
> - 현재로서는 이 방법이 최선인데, 더 효율적인 방법이 있는지 궁금하다.

## 참고한 내용

**파이썬의** `String.strip([chars])`

- String.strip()은 문자열의 양 끝 공백을 모두 제거한다.
- `chars`는 String의 양 옆에서 제거할 문자의 집합이다. 문자열 내 맨 왼쪽 또는 오른쪽 문자와 `chars`의 문자가 일치하지 않으면, 제거과정을 중단한다.
- 예시 코드
    
    ```python
    str = 'abcd'
    print(str.strip('ac'))
    print(str.strip('abc'))
    # 출력 결과
    # bcd	'abcd'에서 'a'만 삭제되었다.
    # d		'abcd'에서 'a','b','c'가 차례대로 삭제되었다.
    
    # 중간에 있는 'c'를 제거하고 싶다면, 끝단에 있는 문자부터 순차적으로 제거해야한다.
    ```