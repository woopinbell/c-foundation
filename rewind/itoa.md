# 정수의 문자열 변환 (itoa)

## `feat(convert): 부호 있는 정수의 문자열 변환 구현`

### 구현

```c
# libft.h

char *ft_itoa(int number);
```

```c
# src/convert/ft_itoa.c

#include "libft.h"
#include <stdlib.h>

static size_t	digit_count(unsigned int magnitude) {
	  size_t count;

	  count = 1;
	  while (magnitude >= 10U) {
        magnitude /= 10U;
        count++;
    }
	  return (count);
}

char	*ft_itoa(int number) {
	  char *text;
	  unsigned int magnitude;
	  size_t length;

	  if (number < 0)
		    magnitude = (unsigned int)(-(number + 1)) + 1U;
	  else
		    magnitude = (unsigned int)number;
	  length = digit_count(magnitude) + (number < 0);
	  text = malloc(length + 1);
    if (text == NULL)
        return (NULL);
    text[length] = '\0';
	  while (length > (size_t)(number < 0)) {
        length--;
        text[length] = (char)('0' + magnitude % 10U);
        magnitude /= 10U;
	  }
	  if (number < 0)
		    text[0] = '-';
	  return (text);
}
```

`ft_itoa`는 `int`를 받아 malloc된 문자열로 변환하는 함수로, 크게 두 단계로 나뉜다.

1. **자릿수 계산 (`digit_count`)**:

    `unsigned int magnitude`를 받아 10으로 나눠가며 자릿수를 센다. 음수/양수를 가리지 않고 순수하게 "크기(magnitude)"만 다룬다.

2. **부호 및 크기 정규화**:
    
    ```c
    if (number < 0)
        magnitude = (unsigned int)(-(number + 1)) + 1U;
    else
        magnitude = (unsigned int)number;
    ```
    단순히 `-number`를 하지 않고 `-(number + 1) + 1U`로 우회한 것이 핵심 설계 판단이다. `INT_MIN`(-2147483648)은 부호를 그대로 뒤집으면 `int` 표현 범위를 벗어나 오버플로우가 나는데, 먼저 `number + 1`로 절댓값을 1 줄여 안전한 범위로 만든 뒤 `unsigned int`로 변환하고 나서 1을 더해 원래 크기를 복원한다. `magnitude`를 처음부터 `unsigned int`로 선언해둔 것도 이 계산을 위한 사전 준비다.

3. **버퍼 할당 및 채우기**:
    
    `length = digit_count(magnitude) + (number < 0)`로 부호 문자 자리까지 포함한 길이를 구하고, `malloc(length + 1)`로 널 종료 문자까지 확보한다. 이후
    ```c
    while (length > (size_t)(number < 0)) {
        length--;
        text[length] = (char)('0' + magnitude % 10U);
        magnitude /= 10U;
    }
    ```
    뒤에서부터 한 자리씩 채워나가는 방식이다. 종료 조건을 `length > 0`이 아니라 `length > (number < 0)`으로 둬서, 음수일 때는 인덱스 0(부호 자리)까지 숫자로 채우지 않고 멈추도록 만들었다. 루프가 끝난 뒤 `if (number < 0) text[0] = '-'`로 부호를 붙이는 순서다.

### 문제 해결 방식
주의할 점은 다. 일반적으로 `itoa`를 단순 구현하면 `-number`가 `int` 범위를 벗어나는 `INT_MIN` 케이스에서 정의되지 않은 동작(UB)이 발생하는데, `magnitude`를 `unsigned int`로 분리하고 `-(number + 1) + 1U`라는 우회 계산을 사용하여 **`INT_MIN` 처리 시의 오버플로우** 문제를 피했다. 또한 `malloc` 실패 시 `NULL`을 즉시 반환하는 방어 분기도 포함되어 있어, 메모리 할당 실패에 대한 기본적인 처리도 함께 처리했다.