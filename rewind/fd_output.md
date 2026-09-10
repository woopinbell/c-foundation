# 파일 디스크립터 출력

## `feat(io): 파일 디스크립터 출력 함수 추가`

```c
# libft.h

void ft_putchar_fd(char c, int fd);
void ft_putstr_fd(char *text, int fd);
void ft_putendl_fd(char *text, int fd);
void ft_putnbr_fd(char n, int fd);
```
```c
# src/io/ft_fd_output.c

#include "libft.h"
#include <unistd.h>

void	ft_putchar_fd(char character, int fd) {
	  (void)write(fd, &character, 1);
}

void	ft_putstr_fd(char *text, int fd) {
	  (void)write(fd, text, ft_strlen(text));
}

void	ft_putendl_fd(char *text, int fd) {
    ft_putstr_fd(text, fd);
    ft_putchar_fd('\n', fd);
}

void	ft_putnbr_fd(int number, int fd) {
    char buffer[11];
    unsigned int magnitude;
    size_t index;

	  if (number < 0)
		    magnitude = (unsigned int)(-(number + 1)) + 1U;
	  else
		    magnitude = (unsigned int)number;
	  index = sizeof(buffer);
    while (magnitude >= 10U) {
        index--;
        buffer[index] = (char)('0' + magnitude % 10U);
        magnitude /= 10U;
	  }
    index--;
    buffer[index] = (char)('0' + magnitude);
    if (number < 0) {
        index--;
        buffer[index] = '-';
    }
	  (void)write(fd, buffer + index, sizeof(buffer) - index);
}
```
`ft_putchar_fd`와 `ft_putstr_fd`는 각각 문자 1개, 문자열을 `write(fd, ...)`로 그대로 내보내는 얇은 래퍼다. `ft_putendl_fd`는 `ft_putstr_fd`로 문자열을 쓴 뒤 `ft_putchar_fd`로 개행을 한 번 더 써서 "줄 단위 출력"을 조립한다.

`ft_putnbr_fd`는 정수를 문자열로 변환하지 않고 11바이트 고정 버퍼(`char buffer[11]`, `int`의 최대 자릿수 10 + 부호 1자리)에 뒤에서부터 채워 넣는 방식을 쓴다. 몫이 남는 동안 `index`를 감소시키며 나머지 자릿수를 채우고, 마지막에 남은 몫(1자리)을 채운 뒤 음수면 `'-'`를 앞에 붙인다. 다 채운 뒤에는 `write(fd, buffer + index, sizeof(buffer) - index)`로 실제 채워진 구간만 한 번에 출력한다. 문자마다 `write`를 호출하지 않고 버퍼링해서 시스템 콜 횟수를 줄이는 선택이었다.

## `fix(io): 파일 디스크립터 출력을 끝까지 재시도`

```c
# src/io/ft_fd_output.c

#define _POSIX_C_SOURCE 200809L
#include "libft.h"
#include <errno.h>
#include <limits.h>
#include <unistd.h>

static int	write_all(int fd, const char *buffer, size_t length) {
    ssize_t written;
    size_t offset;
    size_t request;

    offset = 0;
    while (offset < length) {
        request = length - offset;
        if (request > (size_t)SSIZE_MAX)
            request = (size_t)SSIZE_MAX;
		    written = write(fd, buffer + offset, request);
		    if (written > 0)
			      offset += (size_t)written;
		    else if (written < 0 && errno == EINTR)
			      continue ;
		    else {
			      if (written == 0)
				        errno = EIO;
			      return (0);
		    }
	  }
	  return (1);
}

void	ft_putchar_fd(char character, int fd) {
	  (void)write_all(fd, &character, 1);
}

void	ft_putstr_fd(char *text, int fd) {
	  (void)write_all(fd, text, ft_strlen(text));
}

void	ft_putendl_fd(char *text, int fd) {
	  if (write_all(fd, text, ft_strlen(text)))
		    (void)write_all(fd, "\n", 1);
}

static int	put_unsigned(unsigned int magnitude, int fd) {
	  char	digit;

	  if (magnitude >= 10U && !put_unsigned(magnitude / 10U, fd))
		    return (0);
    digit = (char)('0' + magnitude % 10U);
	      return (write_all(fd, &digit, 1));
}

void	ft_putnbr_fd(int number, int fd) {
	  unsigned int	magnitude;

	  if (number < 0) {
		    if (!write_all(fd, "-", 1))
			      return ;
        magnitude = (unsigned int)(-(number + 1)) + 1U;
	  } else
		    magnitude = (unsigned int)number;
	  (void)put_unsigned(magnitude, fd);
}
```

이전 구현 네 함수가 모두 `write`를 한 번만 호출하고 결과를 버리던 것을 공통 헬퍼 `write_all`로 감싸 안전하게 만든다. 기존 함수들의 시그니처(입출력 인터페이스)는 그대로 두고 내부 구현만 이 헬퍼를 쓰도록 교체하는 리팩터링 작업이다.

새로 추가된 `write_all(int fd, const char *buffer, size_t length)`는 `offset`이 `length`에 도달할 때까지 반복적으로 `write`를 호출한다. 매 반복마다 `length - offset`을 `request`로 쓰되, 이 값이 `SSIZE_MAX`를 넘으면 `SSIZE_MAX`로 잘라서 요청한다. `write`가 `ssize_t`를 반환하므로 요청 크기가 이 범위를 넘으면 반환값 자체가 오버플로될 수 있다는 점을 방어한다.

`write`의 반환값 `written`에 따라 세 갈래로 분기한다: 양수면 그만큼 `offset`을 전진시키고 계속하고, `written < 0`이면서 `errno == EINTR`이면 아무 것도 진행하지 않고 재시도(`continue`)하며, 그 외(음수인데 `EINTR`이 아니거나, 0을 반환한 경우)에는 실패로 보고 `0`을 반환한다. 특히 `written == 0`인 경우를 `errno = EIO`로 명시적으로 세팅해서 실패로 취급하는데, POSIX상 `write`가 0을 반환하는 것은 정상적인 성공 케이스가 아니라고 판단한 것이다.

기존 `ft_putchar_fd`, `ft_putstr_fd`는 단순히 `write` 호출을 `write_all` 호출로 바꾸기만 했다. `ft_putendl_fd`는 이전에는 무조건 문자열을 쓰고 이어서 개행을 썼지만, 이번 수정으로 `if (write_all(text, ...))`로 감싸 문자열 쓰기가 실패하면 개행을 쓰지 않도록 바뀌었다. 앞 단계가 실패하면 뒷단계를 진행하지 않는 순차적 실패 전파를 도입한 것이다.

`put_unsigned`는 재귀적으로 자릿수를 하나씩 `ft_putchar_fd`로 출력하던 방식에서, 각 자리마다 `write_all`을 직접 호출하고 그 성공 여부(`int`)를 반환하도록 바뀌었다. 상위 자릿수 재귀 호출이 실패하면(`!put_unsigned(...)`) 그 즉시 `0`을 반환해 하위 자릿수를 쓰지 않고 중단한다. `ft_putnbr_fd`도 마찬가지로 부호(`'-'`) 쓰기가 실패하면 `return`으로 즉시 빠져나가도록 바뀌었다.

### 문제 해결 방식
`write` 시스템 콜은 요청한 바이트 수보다 적게 쓰고 반환할 수 있고(특히 파이프나 소켓, 큰 버퍼에서), 시그널에 의해 `EINTR`로 중단될 수도 있다는 것이 diff 전체에서 다루는 문제다. `write_all`의 반복 루프와 `EINTR` 시 `continue`가 이를 직접 겨냥한다. 또한 커밋 메시지의 "끝까지 재시도"라는 표현과 `while (offset < length)` 루프가 정확히 대응된다.

부수적으로 `ft_putendl_fd`와 `put_unsigned`/`ft_putnbr_fd`에서 성공 여부를 전파하도록 바꾼 것은, 앞부분 쓰기가 실패했는데 뒷부분을 계속 쓰려고 시도하는 상황(예: 문자열은 못 썼는데 개행만 써지는 것)을 막기 위한 것이다.