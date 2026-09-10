# 인덱스 기반 문자열 변환 (strmapi/striteri 계열)

## 구현 흐름 개요

## `feat(string): 인덱스를 사용하는 문자열 변환 추가`

### 구현

```c
# libft.h

char *ft_strmapi(const char *text, char (*function)(unsigned int, char));
void ft_striteri(char *text, void (*function)(unsigned int, char *));
```

```c
# src/string/ft_string_transform.c

#include "libft.h"
#include <stdlib.h>

char	*ft_strmapi(const char *text, char (*function)(unsigned int, char)) {
    char *mapped;
    size_t length;
    unsigned int index;

	  if (text == NULL || function == NULL)
        return (NULL);
    length = ft_strlen(text);
    mapped = malloc(length + 1);
    if (mapped == NULL)
        return (NULL);
    index = 0;
	  while ((size_t)index < length) {
        mapped[index] = function(index, text[index]);
        index++;
	  }
	  mapped[index] = '\0';
	  return (mapped);
}

void	ft_striteri(char *text, void (*function)(unsigned int, char *)) {
    size_t length;
    unsigned int	index;

	  if (text == NULL || function == NULL)
        return ;
	  length = ft_strlen(text);
	  index = 0;
	  while ((size_t)index < length) {
        function(index, text + index);
        index++;
	  }
}
```

`ft_strmapi`는 원본 문자열 `text`를 한 글자씩 순회하면서 각 인덱스와 문자를 콜백 `function(unsigned int, char)`에 넘겨 그 반환값으로 새 문자열을 만든다. `ft_strlen`으로 길이를 구해 `malloc(length + 1)`로 결과 버퍼를 할당하고, `while` 루프로 `index`를 0부터 `length` 직전까지 돌리며 `mapped[index] = function(index, text[index])`를 채운 뒤 마지막에 널 종단(`mapped[index] = '\0'`)을 붙인다.

`ft_striteri`는 반환값을 만들지 않고 원본 문자열을 직접 수정하는 형태다. 콜백 시그니처가 `void (*)(unsigned int, char *)`로, 문자 값 대신 `text + index`라는 포인터를 넘겨 콜백이 해당 위치를 직접 고칠 수 있게 한다. 두 함수 모두 `text == NULL || function == NULL` 체크를 앞단에 두어 널 포인터 호출을 막는다.

인덱스 변수 `index`는 `unsigned int`로 선언했는데, 길이 비교 시점에는 `size_t`인 `length`와 타입이 맞지 않아 `while ((size_t)index < length)`처럼 비교 쪽에서 캐스팅으로 맞춰주는 방식을 택했다.

### 문제 해결 방식
콜백 함수의 시그니처가 표준 라이브러리(libft) 관례상 인덱스를 `unsigned int`로 받도록 고정되어 있어, 내부 순회 변수도 일단 그에 맞춰 `unsigned int`로 선언했다. 다만 `ft_strlen`의 반환 타입인 `size_t`와 비교할 때 부호/폭 차이가 생기므로, 비교문에서 `(size_t)index`로 캐스팅해 컴파일러 경고를 우회하는 방식으로 처리했다.

### 독백
- `ft_strmapi`의 인덱스 변수를 `unsigned int`로 선언하고 비교 시 `size_t`로 캐스팅하는 방식과, 처음부터 `size_t`로 선언하는 방식 중 어떤 차이가 있다고 생각하나요?
- `ft_striteri`가 값이 아닌 포인터(`char *`)를 콜백에 넘기는 이유는 무엇인가요? `ft_strmapi`와 설계 목적이 어떻게 다른가요?

## 2. `fix(string): callback 순회의 진행 인덱스를 확장`

### 구현

```c
# libft.h
```

```c
#
```


직전 커밋에서 `unsigned int index`와 `(size_t)index` 캐스팅으로 처리했던 두 함수의 순회 변수를, 이 커밋에서는 아예 선언 시점부터 `size_t index`로 바꾸는 방향으로 정리한다.


`ft_strmapi`와 `ft_striteri` 양쪽 모두 동일한 패턴으로 수정된다.

```c
-	unsigned int	index;
+	size_t index;
...
-	while ((size_t)index < length)
+	while (index < length)
- mapped[index] = function(index, text[index]);
+ mapped[index] = function((unsigned int)index, text[index]);
```

순회 변수 자체를 `length`(`size_t`)와 같은 타입인 `size_t`로 바꿔 루프 조건문(`while (index < length)`)에서는 더 이상 캐스팅이 필요 없게 만들었다. 대신 콜백 함수 `function`의 첫 번째 인자 타입이 `unsigned int`로 고정되어 있으므로, 콜백을 호출하는 지점(`function((unsigned int)index, ...)`)으로 캐스팅 위치를 옮겼다.

### 문제 해결 방식
이전 커밋은 "순회 변수 타입"과 "길이 비교"의 타입 불일치를 비교문 쪽에서 임시로 맞췄는데, 이 방식은 `index`가 이론상 `unsigned int`의 최대값을 넘어서는(매우 큰 문자열을 다루는) 경우 순회 카운터 자체가 오버플로될 수 있는 잠재적 문제를 안고 있다. 이 커밋은 순회 변수를 `size_t`로 넓혀 카운터 자체의 표현 범위를 문자열 길이 타입과 일치시키고, 콜백 인터페이스 호환을 위한 캐스팅만 호출 시점으로 최소화했다. 캐스팅을 없앤 것이 아니라 "루프 안전성에 영향 없는 지점"으로 옮긴 것이 이 수정의 핵심이다.