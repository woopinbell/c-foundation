# 정수 파싱 (atoi)

## `feat(convert): 표현 가능한 10진수 정수 해석`

### 구현
```c
# libft.h

int ft_atoi(const char *text);
```

```c
# src/convert/ft_atoi.c

#include "libft.h"
#include <limits.h>

static int	is_space(char c) {
	return (c == ' ' || c == '\t' || c == '\n' || c == '\v' || c == '\f' || c == '\r');
}

int	ft_atoi(const char *text) {
	unsigned int value;
	unsigned int limit;
	int sign;

	while (is_space(*text))
		  ++text;
	sign = 1;
	if (*text == '+' || *text == '-') {
		  if (*text == '-')
			    sign = -1;
		  ++text;
	}
	limit = INT_MAX;
	if (sign < 0)
		  limit = (unsigned int)INT_MAX + 1U;
	value = 0;
	while (*text >= '0' && *text <= '9') {
		  if (value > (limit - (unsigned int)(*text - '0')) / 10U)
			    value = limit;
      else
			    value = value * 10U + (unsigned int)(*text - '0');
		  ++text;
	}
	if (sign < 0 && value == (unsigned int)INT_MAX + 1U)
		  return (INT_MIN);
	return ((int)value * sign);
}
```

`ft_atoi`는 문자열을 정수로 변환하는 함수로, 크게 세 단계로 진행된다.

1. **공백 스킵**: 내부 정적 함수 `is_space`로 `' '`, `'\t'`, `'\n'`, `'\v'`, `'\f'`, `'\r'` 여섯 가지 공백 문자를 판별하고, `while (is_space(*text)) ++text;`로 선행 공백을 모두 건너뛴다. 표준 `isspace`를 직접 쓰지 않고 별도 함수로 구현해 libft 스타일(외부 라이브러리 의존 최소화)을 유지했다.

2. **부호 처리**: `*text`가 `'+'` 또는 `'-'`이면 `sign` 변수(`int`)에 반영하고 포인터를 한 칸 전진시킨다. `sign`은 `1`로 초기화되어 있어 부호가 없는 경우도 자연스럽게 처리된다.

3. **숫자 누적과 오버플로우 방지**: `value`와 `limit`을 모두 `unsigned int`로 선언했다. `sign`이 음수면 `limit`을 `(unsigned int)INT_MAX + 1U`, 즉 `INT_MIN`의 절댓값으로 설정하고, 양수면 `INT_MAX`로 설정한다. 자리마다 `value > (limit - (*text - '0')) / 10U`인지 미리 검사해서, 곱셈/덧셈을 먼저 수행한 뒤 오버플로우 여부를 사후에 확인하는 방식이 아니라 **연산 전에 한계값을 역산해서 넘는지 판단**하는 방식을 택했다. 오버플로우가 나면 그 이후 자리는 계속 `value = limit`으로 고정된다.

마지막으로 `sign < 0 && value == (unsigned int)INT_MAX + 1U`인 경우를 별도로 처리해 `INT_MIN`을 반환한다. 이는 `int`로 캐스팅하기 전에 `unsigned int` 상태에서 정확히 `INT_MIN`의 절댓값과 일치하는지 확인하기 위한 분기로, 부호 있는 정수형의 표현 범위가 비대칭(`INT_MIN`의 절댓값이 `INT_MAX`보다 1 크다)이라는 점을 인지하고 작성한 코드다.

### 문제 해결 방식
이 커밋은 표준 `atoi`가 흔히 겪는 두 가지 문제를 코드 안에서 직접 방어한다.

- **정수 오버플로우**: 단순히 `value * 10 + digit`을 반복하면 `INT_MAX`를 넘는 입력에서 정의되지 않은 동작(부호 있는 정수 오버플로우)이 발생한다. 이 구현은 `unsigned int` 연산으로 오버플로우 자체를 회피하면서, 매 자리마다 한계 초과 여부를 미리 계산해 클램핑(`value = limit`)하는 방식으로 문제를 해결했다.

- **`INT_MIN` 비대칭 문제**: `-2147483648`은 대응하는 양수 `2147483648`이 `int` 범위를 벗어나므로, 단순히 절댓값을 구해서 부호를 곱하는 방식으로는 처리할 수 없다. `limit`을 부호에 따라 다르게 설정하고 `unsigned int` 상태에서 정확한 값을 비교한 뒤 `INT_MIN`을 별도로 반환하는 분기로 이 비대칭성을 정면으로 처리했다.