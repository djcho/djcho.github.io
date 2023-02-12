---
title : "[Java] Reference"
categories:
- Etc
tag :
- [Etc, Java, Reference]
toc: true
toc_sticky : true
published : true
date : 2023-02-12 00:00:00
last_modified_at : 2023-02-12 00:00:00

---



# Java 코딩 테스트 레퍼런스

## 변환

```java
String[] arr = { "A", "B", "C" }; 
// String 배열 -> List로 변환
 List<String> list = Arrays.asList(arr);

//List -> String 배열
String[] arr3 = list.stream().toArray(String[]::new);

// array -> List
int[] arr = { 1, 2, 3 };
List<Integer> intList = Arrays.stream(arr).boxed().collect(Collectors.toList());

// List -> array
int[] arr3 = list.stream().mapToInt(Integer::intValue).toArray();
```



## 정렬

```java
//오름차순 정렬
String arr[] = {"apple","orange","banana","pear","peach","melon"};
Arrays.sort(arr);

//부분 정렬
Arrays.sort(arr, 0, 4); // 0,1,2,3 요소만 정렬

//내림차순 정렬
int arr2[] = {12, 45, 77, 34, 94, 542, 1}
Integer integerArr2 = Arrays.stream(arr2).boxed().toArray(Integer::new);
Arrays.sort(integerArr2, Collections.reverseOrder());

//객체 정렬
Arrays.sort(coffees, new Comparator<Coffee>() {
            @Override
            public int compare(Coffee c1, Coffee c2) {
                return Integer.compare(c1.getPrice(),c2.getPrice());
            }
        });
```



## 구현

```java
//2차원 배열 내의 상하좌우 참조
int dirY[4] = { -1, 1, 0, 0 };
int dirX[4] = { 0, 0, -1, 1 };

int posY = 1, posX = 1;
System.out.println("초기 위치 Y :" + posY + " X : " + posX);

for (int i = 0; i < 4; i++) {
		int nextY = posY + dirY[i];
		int nextX = posX + dirX[i];
  	System.out.println("Y : " + nextY + " X : " + nextX);
}

//파일 입력
BufferedReader reader = new BufferedReader(new FileReader(filename));
String line = null;
while( (line = reader.readLine()) != null ){
  	System.out.println(line);
}

//파일 출력
PrintWriter writer = new PrintWriter("copy_" + filename);
writer.println(file);
writer.close();

//패턴 검색
Pattern pattern = Pattern.compile("(?<=[/\\\\])[^/\\\\]+\\.(gif|GIF)"); //문자열 중 gif혹은 GIF 파일 검출
Matcher matcher = pattern.matcher(inputString);
if (matcher.find()) {
    System.out.println(matcher.group());
}
```

