---
title: how to use markdown
date: 2019-01-12 22:27:47
tags:
- markdown
categories:
- markdown
---

markdown으로 글쓰기

### 제목을 지정해보자

```
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

### 링크 첨부하기

```
[Link1](https://j2yes.github.io/)

[j2yess@gmail.com](mailto:j2yess@gmail.com)
```
> [Link1](https://j2yes.github.io/)

> [j2yess@gmail.com](mailto:j2yess@gmail.com)

### 이미지 첨부하기

```
![background](/images/jenkins/1.png "Optional title")
```

![background](/images/jenkins/1.png "Optional title")

### 파일(PDF) 첨부하기

```
[다운로드](/assets/mydoc.pdf)
```

> [다운로드](/assets/mydoc.pdf)

### 코드(GIST) 첨부하기

```
{% gist j2yes/3451d495ee5b9a601385 %}
```
{% gist j2yes/3451d495ee5b9a601385 %}

### 코드블럭 첨부하기

```sbtshell
ps -ef | grep oh
```

```javascript
console.log('oh');
```

```java
static void main(String args[]) {
    
}
```
    

### 수평선으로 구분하기

```
---
or
***
or
___
```

### 번호 없는 리스트

```
+ Java
- Python
* Ruby
```

+ Java
- Python
* Ruby

### 번호 있는 리스트

```
1. 첫번째
2. 두번째
3. 세번째
```

1. 첫번째
2. 두번째
3. 세번째

### 체크박스 

```
- [ ] 리스트1
- [X] 리스트2
```

- [ ] 리스트1
- [X] 리스트1

### 텍스트에 효과주기

```
**두껍게**
~~취소선~~
```

기본글씨

**두껍게**

~~취소선~~

### \` 사용하기

```
\`
```

> \`

### 음영효과주기

```
`keyword`
```

`keyword`


### Table
| Item      |    Value | Qty  |
| :-------- | --------:| :--: |
| Computer  | 1600 USD |  5   |
| Phone     |   12 USD |  12  |
| Pipe      |    1 USD | 234  |


### 참고사이트
* https://sourceforge.net/p/collapse/wiki/markdown_syntax/
* http://commonmark.org/help/tutorial/index.html
* https://guides.github.com/features/mastering-markdown/
* http://marxi.co/