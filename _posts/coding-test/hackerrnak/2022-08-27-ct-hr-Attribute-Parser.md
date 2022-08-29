---
title : "[HackerRank][String]Attribute Parser"
categories:
- HackerRank
tag :
- [HackerRank, Algoritm, Programming, C++]
toc: true
toc_sticky : true
published : true
date : 2022-08-27
last_modified_at : 2022-08-27
---

## 🔍문제

This challenge works with a custom-designed markup language *HRML*. In *HRML*, each element consists of a starting and ending tag, and there are attributes associated with each tag. Only starting tags can have attributes. We can call an attribute by referencing the tag, followed by a tilde, '`~`' and the name of the attribute. The tags may also be nested.

The *opening tags* follow the format:

```
<tag-name attribute1-name = "value1" attribute2-name = "value2" ...>
```

The *closing tags* follow the format:

```
</tag-name>
```

The attributes are referenced as:

```
tag1~value  
tag1.tag2~name
```

Given the source code in HRML format consisting of lines, answer queries. For each query, print the value of the attribute specified. Print *"Not Found!"* if the attribute does not exist.

### **Example**

```
HRML listing
<tag1 value = "value">
<tag2 name = "name">
<tag3 another="another" final="final">
</tag3>
</tag2>
</tag1>

Queries
tag1~value
tag1.tag2.tag3~name
tag1.tag2~value
```

Here, tag2 is nested within tag1, so attributes of tag2 are accessed as `tag1.tag2~<attribute>`. Results of the queries are:

```
Query                 Value
tag1~value            "value"
tag1.tag2.tag3~name   "Not Found!"
tag1.tag2.tag3~final  "final"
```

### **Input Format**

The first line consists of two space separated integers, and . specifies the number of lines in the HRML source program. specifies the number of queries.

The following lines consist of either an opening tag with zero or more attributes or a closing tag. There is a space after the tag-name, attribute-name, '=' and value.There is no space after the last value. *If there are no attributes there is no space after tag name.*

 queries follow. Each query consists of string that references an attribute in the source program.More formally, each query is of the form ~ where and are valid tags in the input.

### **Constraints**

- Each line in the source program contains, at most, characters.
- Every reference to the attributes in the queries contains at most characters.
- All tag names are unique and the HRML source program is logically correct, i.e. valid nesting.
- A tag can may have no attributes.

### **Output Format**

Print the value of the attribute for each query. Print "*Not Found!*" without quotes if the attribute does not exist.

### **Sample Input**

```
4 3
<tag1 value = "HelloWorld">
<tag2 name = "Name1">
</tag2>
</tag1>
tag1.tag2~name
tag1~name
tag1~value
```

### **Sample Output**

```
Name1
Not Found!
HelloWorld
```

  

## 📝내 풀이

### 코드

```c++
//Attribute Parser in C++ - Hacker Rank Solution
#include <cmath>
#include <cstdio>
#include <vector>
#include <iostream>
#include <algorithm>
#include <bits/stdc++.h>
using namespace std;

class Tag
{
private:
    std::string tagName;
    std::map<std::string, std::string> attributes;
    std::vector<Tag*> childTags;
    Tag* parentTag;

public:

    void setParent(Tag* parentTag)
    {
        this->parentTag = parentTag;
    }

    Tag* getParentTag()
    {
        return this->parentTag;
    }

    std::string getName()
    {
        return this->tagName;
    }

    Tag(std::string tagName, std::map<std::string, std::string> attributes)
        :tagName(tagName), attributes(attributes)
    {
    }

    std::map<std::string, std::string>& getAttributes() { return this->attributes; }
    void addChildTag(Tag* tag)
    {
        childTags.push_back(tag);
    }

    Tag* getChildByTagName(const std::string& tagName)
    {
        std::vector<Tag*>::iterator pos = std::find_if(this->childTags.begin(), this->childTags.end(), [&](Tag* tag) {
            return tag->getName() == tagName; });

        if (pos != this->childTags.end())
            return *pos;
        return nullptr;
    }
};

class StringUtil
{
public:

    static std::string replace(const std::string& src, const std::string& from, const std::string& to) {

        std::string replaced = src;
        size_t start_pos = 0;
        while ((start_pos = replaced.find(from, start_pos)) != std::string::npos) {
            replaced.replace(start_pos, from.length(), to);
            start_pos += to.length();
        }
        return replaced;
    }

    static std::vector<std::string> splitString(const std::string& src, const std::string& delimiter)
    {
        std::vector<std::string> vec;
        std::string tmp_src = src;
        std::size_t pos;
        while ((pos = tmp_src.find(delimiter)) != std::string::npos)
        {
            std::string splitStr = tmp_src.substr(0, pos);
            vec.push_back(splitStr);

            tmp_src.erase(0, pos + delimiter.length());
        }
        vec.push_back(tmp_src);
        return vec;
    }
};

class HrmlParser
{
private:
    std::vector<std::string> sourceCode;
    std::vector<Tag*> tags;

public:
    HrmlParser(std::vector<std::string> sourceCode) : sourceCode(sourceCode) {}
    ~HrmlParser() {}

    std::string query(const std::string queryString)
    {
        std::vector<std::string> splitedQueryString = StringUtil::splitString(queryString, "~");
        std::vector<std::string> splitedTagNames = StringUtil::splitString(splitedQueryString[0], ".");
        std::string attributeName = splitedQueryString[1];

        Tag* currentTag = nullptr;
        std::vector<Tag*>::iterator tagPos = std::find_if(this->tags.begin(), this->tags.end(), [&](Tag* tag)
            {
                return tag->getName() == splitedTagNames[0];
            });

        if(tagPos == this->tags.end())
            return std::string("Not Found!");

        currentTag = *tagPos;

        if (splitedTagNames.size() >= 2)
        {
            for (std::vector<std::string>::iterator iter = splitedTagNames.begin() + 1; iter != splitedTagNames.end(); iter++)
            {
                currentTag = currentTag->getChildByTagName(*iter);
                if(!currentTag)
                    return std::string("Not Found!");
            }
        }

        std::map<std::string, std::string> attributes = currentTag->getAttributes();
        std::map<std::string, std::string>::iterator pos = attributes.find(attributeName);
        if (pos == attributes.end())
            return std::string("Not Found!");
        else
            return pos->second;
    }


    std::string getTagName(const std::string& tagString)
    {
        //<tag1 value = "HelloWorld">
        std::string temp = tagString;
        temp.erase(std::remove(temp.begin(), temp.end(), '<'), temp.end());
        temp.erase(std::remove(temp.begin(), temp.end(), '>'), temp.end());
        size_t found = temp.find(' ');
        if (found == std::string::npos)
            return temp;

        return  temp.substr(0, tagString.find(' ') - 1);;
    }

    std::map<std::string, std::string> getAttributes(const std::string& tag)
    {
        //<tag3 another = "another" final = "final">
        std::map<std::string, std::string> attributes;

        std::size_t found = tag.find(' ');
        if (found == std::string::npos)
           return attributes;

        std::string attributeStrings = tag.substr(found + 1, tag.find_last_of('>') - (found + 1));
        
        std::vector<std::string> splitedAttributes = StringUtil::splitString(attributeStrings, "\" ");
        
        
        for (auto& splitedAttribute : splitedAttributes)
        {
            splitedAttribute.erase(std::remove(splitedAttribute.begin(), splitedAttribute.end(), '\"'), splitedAttribute.end());
            std::vector<std::string> attKeyValue = StringUtil::splitString(splitedAttribute, " = ");
            attributes.insert(std::pair<std::string, std::string>(attKeyValue[0], attKeyValue[1]));
        }

        return attributes;
    }

     Tag* createTag(const std::string& tagString)
     {
         return new Tag(getTagName(tagString), getAttributes(tagString));
     }

     void parse()
     {
         Tag* currentTag = nullptr;
         for (auto& tagString : this->sourceCode)
         {
             if (isCloser(tagString))
             {
                 currentTag = currentTag->getParentTag();
                 continue;
             }

             Tag *newTag = createTag(tagString);
             if (currentTag)
                 currentTag->addChildTag(newTag);  

             newTag->setParent(currentTag);
             currentTag = newTag;

             if(!currentTag->getParentTag())
                tags.push_back(newTag);
         }
     }
    
    bool isCloser(const std::string& tag)
    {
        if (tag.empty())
            return false;

        std::size_t found = tag.find("</");
        if (found != std::string::npos)
            return true;
        else
            return false;
    }
};


std::vector<std::string> inputCode(int inputLine)
{
    std::string lineOfCode;
    std::vector<std::string> intputData;

    for (int i = 0; i < inputLine; ++i)
    {
        std::getline(std::cin, lineOfCode);
        intputData.push_back(lineOfCode);
    }

    return intputData;
}

int main() {
    /* Enter your code here. Read input from STDIN. Print output to STDOUT */
    int sourceLine = 0;
    int queryLine = 0;

    std::cin >> sourceLine >> queryLine;
    std::cin.ignore();

    std::vector<std::string> sourceCode = inputCode(sourceLine);
    std::vector<std::string> queryCode = inputCode(queryLine);

    HrmlParser hrmlParser(sourceCode);
    hrmlParser.parse();
    for (auto& query : queryCode)
        std::cout << hrmlParser.query(query) << std::endl;

    return 0;
}
```



<br>

### 풀이

문제를 다 풀고 나서 다른 사람들의 답안을 몇 개를 봤는데 클래스를 정의해서 푼 사람을 그리 많지 않았다. 클래스를 정의하지 않고 푼 코드는 아무래도 내 코드와는 다르게 코드 라인이 적고 불필요한 반복문은 존재하지 않아 성능 상의 이점이 존재했다. 하지만 알고리즘 풀이 답에 가독성이 뭐가 중요하냐만은 코드의 가독성과 구조는 역시 좋지 좋지 않았다. 나는 왜 이런 결과 지향적이고 풀이 시간에 효율적인 코드를 생각하지 못했는지는 아마도 알고리즘적인 사고를 잘 하지 못했던 것 같다. 반대로 내 풀이는 성능에 있어서는 약간 내어주더라도 클래스의 시그니처만으로 처음 이 코드를 접하는 사람에게 전체적인 흐름을 유추 또는 파악할 수 있다는 것이 장점이다. 풀이는 아래와 같다.



#### HrmlParser

일단 문제의 핵심은 Hrml 이라는 소스코드를 파싱 하는 것이기 때문에 `HrmlParser`를 정의하였다. `HrmlParser`의 핵심 메서드는 주어진 소스코드를 입력으로 받아 인스턴스를 생성하고 내부 자료구조에 파싱된 데이터를 인스턴스화시키는 `parser()` 메서드와 정해진 규칙에 따라 입력되는 QrueyString을 파싱하고 결과를 출력하는 `query()`  메서드이다.
또 위 메서드 외에 태그 이름을 구하는 `getTagName()`메서드와 어트리뷰트의 목록을 파싱하는 `getAttributes()` , Closer 인지를 확인하는`isCloser()` 메서드들이 존재한다.



 메인 파싱 로직은 최초 입력 받은 소스코드를 한줄씩 읽으면서 새로 만나는 태그들의 객체를 생성하고 이 객체의 부모와 자식 관계를 연결한다. `query()` 메서드에서는 "`~`" 로 Split 하여 태그의 위치를 "`.`"으로 표현하는 위치 부분과 쿼리 대상 어트리뷰트의 이름을 나누고, 이 데이터를 가지고 내부 자료구조들을 탐색한다.



#### Tag

Hrml 의 소스는 "`<`"로 시작하는 태그로 구성되어 있다. 이 태그들의 특징은 하이어라키 관계로 중첩되며 태그 이름과 어트리뷰트의 목록으로 구성된다. 그리하여`Tag`클래스에는 `tagName` 과 `attibutes`,  하이어라키 관계를 코드에 녹일 수 있도록 부모 태그와 자식 태그들의 포인터를 포함한다.

