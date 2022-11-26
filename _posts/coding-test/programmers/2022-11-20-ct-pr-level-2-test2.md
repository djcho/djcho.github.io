---
title : "[Programmers][Lv.2]JadenCase 문자열 만들기"
categories:
- Programmers
tag :
- [Programmers, Algoritm, Programming, Java]
toc: true
toc_sticky : true
published : true
date : 2022-11-20 00:00:00
last_modified_at : 2022-11-20 00:00:00
---

## 🔍문제

JadenCase란 모든 단어의 첫 문자가 대문자이고, 그 외의 알파벳은 소문자인 문자열입니다. 단, 첫 문자가 알파벳이 아닐 때에는 이어지는 알파벳은 소문자로 쓰면 됩니다. (첫 번째 입출력 예 참고)
문자열 s가 주어졌을 때, s를 JadenCase로 바꾼 문자열을 리턴하는 함수, solution을 완성해주세요.

##### 제한 조건

- s는 길이 1 이상 200 이하인 문자열입니다.
- s는 알파벳과 숫자, 공백문자(" ")로 이루어져 있습니다.
  - 숫자는 단어의 첫 문자로만 나옵니다.
  - 숫자로만 이루어진 단어는 없습니다.
  - 공백문자가 연속해서 나올 수 있습니다.

##### 입출력 예

|            s            |         return          |
| :---------------------: | :---------------------: |
| "3people unFollowed me" | "3people Unfollowed Me" |
|   "for the last week"   |   "For The Last Week"   |



## 📝내 풀이

##### 코드

```java
class Solution {
    public String solution(String s) {
        String answer = "";
        boolean blank = true;

        String[] split = s.toLowerCase().split("");
        for (String str : split){
            answer += blank ? str.toUpperCase() : str;
            blank = str.equals(" ") ? true : false;
        }
        return answer;
    }
}
```
