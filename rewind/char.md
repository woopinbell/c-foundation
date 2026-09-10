# 문자 판별과 대소문자 변환 (ctype)

## `feat(char): ASCII 문자 판별과 대소문자 변환 구현`
`is*` 판별 함수군과 `toupper`/`tolower` 변환을 구현

### 구현
```c
# libft.h

#ifndef LIBFT_H
# define LIBFT_H

int	ft_isalpha(int c);
int	ft_isdigit(int c);
int	ft_isalnum(int c);
int	ft_isascii(int c);
int	ft_isprint(int c);
int	ft_toupper(int c);
int	ft_tolower(int c);

#endif
```

```c
# src/char/ft_char.c

#include "libft.h"

int	ft_isalpha(int c) {
	return ((c >= 'A' && c <= 'Z') || (c >= 'a' && c <= 'z'));
}

int	ft_isdigit(int c) {
	return (c >= '0' && c <= '9');
}

int	ft_isalnum(int c) {
	return (ft_isalpha(c) || ft_isdigit(c));
}

int	ft_isascii(int c) {
	return (c >= 0 && c <= 127);
}

int	ft_isprint(int c) {
	return (c >= 32 && c <= 126);
}

int	ft_toupper(int c) {
	  if (c >= 'a' && c <= 'z')
		    return (c - ('a' - 'A'));
	  return (c);
}

int	ft_tolower(int c) {
	  if (c >= 'A' && c <= 'Z')
		    return (c + ('a' - 'A'));
    return (c);
}
```

빌드 인프라(`.gitignore`, `Makefile`)와 함께 libft의 `ctype.h` 계열 함수 7개(`ft_isalpha`, `ft_isdigit`, `ft_isalnum`, `ft_isascii`, `ft_isprint`, `ft_toupper`, `ft_tolower`)를 `src/char/ft_char.c` 한 파일에 구현, 각 함수는 표준 `ctype.h` 함수와 동일한 시그니처(`int c` 입력, `int` 반환)를 따르며, 구현 방식은 라이브러리 함수 호출 없이 문자 범위를 직접 비교하는 방식이다.

- `ft_isalpha`: `'A'~'Z'` 또는 `'a'~'z'` 범위 비교의 논리합
- `ft_isdigit`: `'0'~'9'` 범위 비교
- `ft_isalnum`: `ft_isalpha`와 `ft_isdigit`를 조합해서 새 로직을 만들지 않고 이미 만든 함수를 재사용하여 구현
- `ft_isascii`: `0~127` 범위 비교로, ASCII 전체 범위를 다룬다는 점에서 `isprint`(출력 가능 문자만)와 역할 분리
- `ft_isprint`: `32~126` 범위 비교, 공백(32)부터 물결표(126)까지의 출력 가능 문자 범위
- `ft_toupper`/`ft_tolower`: 조건에 맞는 범위일 때만 `'a' - 'A'`(32)만큼 더하거나 빼고, 해당하지 않으면 입력값을 그대로 반환, ASCII 문자 코드값에 의존하지 않고 문자 리터럴 간의 차이로 계산하도록 구현

### 문제 해결 방식
`ft_isalnum`이 `ft_isalpha`/`ft_isdigit`를 호출하는 구조로 이후 문자 판별 로직이 바뀌더라도(예: 범위 조건 수정) 한 곳만 고치면 되도록 구현했다.