---
title : "자료구조의 종류와 각 종류에 대한 설명"
categories : 
    - DataStructure
tag :
    - DataStructure
toc : true
---

# 자료구조의 종류와 각 종류에 대한 설명


## 자료구조란

그릇 (자료)을 논리적으로 정의된 규칙에 의해 효율적으로 관리하는 방법

## 자료구조의 종류

![error](/assets/images/study/TTL/2022-09-05/01-data-structure.png)  

자료구조의 종류로는 자료형의 따라 분류하는 단순 구조와 선형 구조, 비선형 구조, 파일 구조가 있다.

## 단순구조

단순구조 자료구조는 정수, 실수, 문자, 문자열과 같은 컴퓨터가 기본적으로 제공하는 자료형을 뜻한다.

### 정수(Integer)

소수점이 없는 형태로,  0, 1, 10, 100, -100 과 같은 숫자를 의미한다.

### 실수 (Float)

소수점이 있는 형태이다.

### 문자 (Character)

한 글자를 의미한다. 일반적으로 프로그래밍 언어에서는 작은 따옴표를 통해 표기한다.

### 문자열 (String)

문자 여러개를 묶은 것으로 일반적으로 프로그래밍 언어에서는 큰 따옴표를 통해 표기한다.

## 선형 구조

데이터를 순차적으로 한 줄로 표기한 자료구조로 선형리스트, 연결리스트, 스택, 큐가 있다.

### 선형리스트 (Linear List)

데이터를 논리적인 순서대로 메모리에 연속하여 저장하는 구현 방식을 의미하며,

주로 고정된 크기를 가진 배열을 통해 구현한다.

그렇기 때문에 각 원소의 순서가 정해져 있으며, 각 원소의 인덱스를 사용할 수 있다.

이로인해 데이터 검색에 효율적이지만, 삽입과 삭제 시 크기를 더 할당하거나, 모든 데이터를 이동해줘야하기 때문에 비효율적이다.

### 연결리스트 (Linked List)

연결리스트는 노드를 통해 연결하는 리스트를 말한다.

선형리스트와 달리 고정이 아닌, 가변적인 크기의 리스트를 가진다.

따라서 데이터를 삽입, 제거 시 크기 할당이나, 데이터를 이동할 필요 없이 다음 노드를 추가하거나, 제거할 수 있다.

다만, 검색에 대해서는 선형리스트와 다르게 원하는 데이터를 한번에 찾아낼 수 없고, 리스트를 전부 탐색해야 한다.

주로 데이터 삽입, 제거가 많이 발생하는 프로그램이라면 연결리스트가 유리하고, 탐색이 주가 되는 프로그램이라면 선형리스트를 사용하는 것이 유리하다.

### 스택 (Stack)

![error](/assets/images/study/TTL/2022-09-05/02-stack.png)  

스택은 한쪽으로만 데이터를 넣고 뺄 수 있는 구조의 자료구조이다.

그렇기 때문에 마지막에 넣은 데이터가 가장 먼저 나가는 LIFO (Last-In First-Out) 구조를 가진다.

주로 연결리스트를 통해 구현하고, 웹브라우저의 페이지 뒤로가기, undo, 문자열 뒤집기와 같은 프로그램에 사용된다.

### 큐 (Queue)

![error](/assets/images/study/TTL/2022-09-05/03-queue.png)    

큐는 양쪽으로 데이터를 넣고 뺄 수 있는 자료구조이다.

큐의 종류에 따라 차이점이 있지만, 기본적인 큐는 한쪽으로 데이터를 넣고, 다른 한쪽으로 데이터를 뺄 수 있다.

따라서 가장 먼저 들어온 데이터가 가장 먼저 나오는 FIFO (First-In First-Out) 구조를 가진다.

큐는 일반적으로 연결리스트를 통해 구현하며, 선입선출을 필요로 하는 대기열, 프로세스 관리, 너비 우선 탐색에 사용된다.

## 비선형 구조

비선형 구조는 하나의 자료 뒤에 여러개의 자료가 존재할 수 있는 자료구조이다.

트리와 그래프가 대표적인 비선형 구조의 자료구조이다.

### 트리 (Tree)

![error](/assets/images/study/TTL/2022-09-05/04-tree.png)  

![error](/assets/images/study/TTL/2022-09-05/05-tree2.png)  

트리는 노드가 나무 가지처럼 연결되어 있는 비선형 자료구조이다.

마치 나무를 뒤집어 놓은 모습과 유사하다.

트리 내 또 다른 하위 트리가 있고, 그 안에 또 다른 하위 트리가 있는 재귀적인 구조를 가진다.

컴퓨터의 디렉토리 구조가 트리 구조이다.

### 그래프 (Graph)

![error](/assets/images/study/TTL/2022-09-05/06-graph.png)  

그래프는 노드와 노드를 연결하는 간선을 모아놓은 자료구조이다.

트리와 많이 비교가 되는데 트리 역시 그래프의 형태이나, 그래프는 방향이 있거나, 없을 수 있으며,

루트 노드라는 개념이 없다. 또한 사이클이 가능한 형태로 구성되어 있다.

## 파일 자료구조

파일이 디스크에 저장되는 방식에 따라 순차 파일과, 직접 파일, 색인 순차 파일로 구분된다.

### 순차 파일 (S**equential File)**

파일 내용을 논리적인 처리 순서에 따라 연속해서 저장하는 것을 말한다.

구조가 간단하기 때문에 공간 효율성은 높지만, 다른 내용을 추가하거나 제거할 때 

파일 내용을 재구성 해야하기 때문에 시간이 오래걸린다.

### 직접 파일 (D**irect File)**

파일의 내용을 임의의 물리적인 위치에 기록하는 방식이다.

내용을 저장하는 위치는 해시 함수 라는 계산식으로 결정된다.

### 색인 순차 파일 (**Indexed Sequential File**)

순차 파일과 직접 파일이 결합된 형태로, 데이터를 키 값 순서로 정렬시켜 기록하고, 

키 항목 만을 모은 색인을 구성하여 편성하는 방식이다.