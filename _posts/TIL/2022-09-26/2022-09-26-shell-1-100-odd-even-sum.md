---
title : "리눅스 Shell Script 1부터 100까지 짝수, 홀수 더하기"
categories : 
    - linux
tag :
    - shell script
toc : true
---

# 리눅스 Shell Script 1부터 100까지 짝수, 홀수 더하기


## 프로그램 동작 방식

어떤 방법으로 짝수와 홀수를 더할까 고민하다가,

프로그램을 실행하고 그냥 두 개의 값이 모두 나와버리면 재미가 없을 것 같았다.

사용자로부터 값을 받아 짝수를 더할지, 홀수를 더할지 정하기로 했다.

## 스크립트 지정

먼저 셸 스크립트를 작성하려면 어떤 스크립트를 사용할지 지정해줘야 한다.

sh와 bash가 대표적인데, sh를 사용해보았다.

```sh
#!/bin/sh
```

## 값 입력

값을 입력 받을때는 `read`를 사용하는데, 보통 `echo`나 `printf`를 통해 받아온다.

echo는 줄바꿈을 하고, printf는 줄바꿈을 하지 않는다.

나는 줄바꿈을 하지 않고

`This is an odd, even sum program... input odd or even (default : odd) >> even`

이런 식으로 출력하고 싶어서 printf를 사용했다.

```sh
printf 'This is an odd, even sum program... input odd or even (default : odd) >> '
read TYPE
```

## Default 값 설정

default로 홀수를 받도록 했는데, 공백을 입력 받았을 때 홀수 처리를 하도록 작성했다.

```sh
if [ -z $TYPE ]; then
    TYPE='odd'
fi
```

if문에서 `-z`는 변수가 null인지 체크한다.

참고로 `-n`은 변수가 null이 아닌지 체크한다.

## For Loop 시작 값 설정

이제 For Loop 시작 값을 세팅해줘야 한다. 

odd를 입력 받았다면 시작 값을 1로 세팅하고, even을 받았다면 시작 값을 2로 세팅, 

그리고 다른 값을 받았을 때는 잘못된 값을 입력했다는 말을 출력하고 프로그램을 종료한다.

```sh
if [ $TYPE == 'odd' ]; then
    START_NUM=1
elif [ $TYPE == 'even' ]; then
    START_NUM=2
else 
    echo $TYPE 'is not a valid value. input odd or even (default : even)'
    exit
fi
```

## For Loop

마지막으로 For Loop을 돌릴 건데 `for in`문을 사용하기로 했다.

for in문은 범위를 설정할 수 있는데 `seq` 키워드를 사용하면 연속적인 수를 사용한다는 의미이다.

seq 뒤로 순서대로 시작값, 증가값, 종료값을 파라미터를 줄 수 있다.

즉 `for i in $(seq 1 2 100)` 을 사용한다면, 1부터 100까지 loop을 돌리는데, 2씩 증가한다는 의미이다.

이후 Loop을 돌면서 각각의 element를 더해준다.

```sh
for i in $(seq $START_NUM 2 100)
do
    SUM=$(($SUM+$i))
done
```

## 최종 값 출력

최종 값을 출력하고 프로그램은 종료된다.

```sh
echo $TYPE 최종 합계 = $SUM
```

## 최종 코드

최종적으로 작성된 프로그램은 아래와 같다.

```sh
#!/bin/sh

# Input 읽기 
printf 'This is an odd, even sum program... input odd or even (default : odd) >> '
read TYPE

# Input 값이 없다면 홀수 설정
if [ -z $TYPE ]; then
    TYPE='odd'
fi

# 짝수/홀수 시작값을 설정하고 다른 값이 들어오면 종료
if [ $TYPE == 'odd' ]; then
    START_NUM=1
elif [ $TYPE == 'even' ]; then
    START_NUM=2
else 
    echo $TYPE 'is not a valid value. input odd or even (default : even)'
    exit
fi

# 시작값 부터 100까지 2씩 증가하며 Loop
for i in $(seq $START_NUM 2 100)
do
    # Loop 돌면서 합계 구하기
    SUM=$(($SUM+$i))
done

# 최종 값 출력
echo $TYPE 최종 합계 = $SUM
```
