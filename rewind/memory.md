# 메모리 함수

## 1. `feat(memory): 메모리 채우기와 0 초기화 구현`

### 구현
```c
# libft.h

# include <stddef.h>

void *ft_memset(void *memory, int byte, size_t length);
void ft_bzero(void *memory, size_t length);
```

```c
# src/memory/ft_memory_fill.c

#include "libft.h"

void	*ft_memset(void *memory, int byte, size_t length) {
	  unsigned char	*byte_pointer;

	  byte_pointer = memory;
	  while (length > 0) {
        *byte_pointer = (unsigned char)byte;
        ++byte_pointer;
        --lengthㄴ;
	  }
	  return (memory);
}

void	ft_bzero(void *memory, size_t length) {
	  ft_memset(memory, 0, length);
}
```

`libft.h`에 `size_t`를 쓰기 위해 `#include <stddef.h>`를 추가했다.

`ft_memset`은 `void *`로 받은 메모리를 `unsigned char *byte_pointer`로 캐스팅해 1바이트씩 순회하며 `(unsigned char)byte` 값을 채워 넣는 방식이다. 포인터 산술이 바이트 단위로 이뤄지도록 `unsigned char` 타입을 사용. `length`를 감소시키며 `0`이 될 때까지 반복하는 단순 `while` 루프 구조를 취했고, 끝에는 원본 `memory` 포인터를 반환해 원본 libc `memset`과 동일한 시그니처/반환 관례를 적용했다.

`ft_bzero`는 `0`으로 채우는 것이 값 채우기의 특수 케이스라는 점을 활용해 별도 로직 없이 `ft_memset(memory, 0, length)`를 그대로 호출하는 얇은 래퍼로 구현하여 중복구현을 피했다.

### 문제 해결 방식
libc의 `memset`의 관례대로 상위 바이트를 버리고 하위 1바이트만 사용한다는 규약을 지키기 위해 `byte` 매개변수가 `int`로 들어오는 것을 `(unsigned char)`로 캐스팅해 저장했다.

## 2. `feat(memory): 겹치지 않는 메모리 복사 구현`

## 구현
```c
# libft.h

void *ft_memcpy(void *dest, const void *src, size_t length)
```

```c
# src/memory/ft_memory_copy.c

#include "libft.h"

void *ft_memcpy(void *dest, const void *src, size_t length) {
    unsigned char *dest_byte;
    const unsigned char *src_byte;

    dest_byte = dest;
    src_byte = src;
    while (length > 0) {
        dest_byte = *src_byte;
        ++dest_byte;
        ++src_byte;
        --length;
    }
    return (dest);
}
```

앞서 구현한 `ft_memset` 기반의 바이트 단위 포인터 순회 패턴을 그대로 가져와 읽기 전용 소스에서 목적지로 복사하는 `ft_memcpy`를 구현했다.

`dest`은 `unsigned char *`로, `src`는 `const unsigned char *`로 각각 캐스팅해 두 포인터를 동시에 증가시키며 `1byte`씩 대입하는 구조이며 `src`를 `const`로 선언하여 원본 데이터를 수정하지 않는다는 의도를 타입 시스템으로 보장했다. `ft_memset`과 마찬가지로 `length`가 `0`이 될 때까지 `while` 루프를 돌고, 끝에 `dest`을 반환한다.

## 3. `feat(memory): 겹치는 메모리의 안전한 이동 구현`

### 구현
```c
# libft.h

void *ft_memmove(void *dest, const void src, size_t length);
```

```c
# src/memory/ft_memory_move.c

#include "libft.h"

void *ft_memmove(void *dest, const void *src, size_t length) {
	  unsigned char *dest_byte;
	  const unsigned char *src_byte;
	  size_t offset;

	  dest_byte = dest;
	  src_byte = src;
	  if (dest_byte == src_byte || length == 0)
	  	  return (dest);
	  offset = 1;
	  while (offset < length) {
        if (dest_byte == src_byte + offset) {
            while (length > 0) {
                --length;
                dest_byte[length] = src_byte[length];
            }
            return (dest);
        }
        ++offset;
	  }
	  ft_memcpy(dest_byte, src_byte, length);
	  return (dest);
}
```

`ft_memcpy`가 처리하지 못하는 겹치는 메모리 영역 이동 문제를 해결하기 위해 추가 구현한 함수로 내부에서 `ft_memcpy`를 호출해 겹치지 않는 경우를 위임하는 구조이다.

`dest_byte`와 `src_byte`가 같거나 `length`가 `0`이면 아무 작업 없이 바로 반환하는 얼리 리턴한다. 이후 `offset`을 1부터 증가시키며 `dest_byte == src_byte + offset`인지, 즉 목적지가 소스보다 뒤쪽에 있으면서 겹치는지를 검사하는 `while` 루프를 돌며 조건이 참이면 뒤쪽(`length - 1`)부터 역순으로 복사하는 내부 `while` 루프를 실행해, 이미 읽어야 할 값을 덮어쓰기 전에 먼저 읽도록 만든다. 겹치지 않는 것으로 판단되면(`offset`이 `length`에 도달할 때까지 겹침이 발견되지 않으면) 마지막에 `ft_memcpy`를 그대로 호출해 정방향 복사로 처리한다.

### 문제 해결 방식
소스와 목적지 메모리 영역이 겹칠 때 정방향 복사(`ft_memcpy`)를 그대로 쓰면 아직 읽지 않은 원본 데이터가 손상된다. 이를 `offset`을 이용해 목적지가 소스 뒤쪽에 있는 겹침 패턴(`dest_byte == src_byte + offset`)을 탐지하고, 해당 경우에만 역순 복사로 전환해 데이터 손상을 막도록 구현했다. 다만 이 겹침 탐지 로직은 `offset`을 1부터 `length`까지 순회하며 매번 포인터를 비교하는 방식이라, 일반적인 `dest > src` 포인터 비교 방식에 비해 시간 복잡도 측면에서는 손해다.

## 4. `feat(memory): 범위를 제한한 메모리 검색과 비교 추가`

## 구현
```c
# libft.h

void *ft_memchr(const void *memory, int byte, size_t length);
int ft_memcmp(const void *left, const void *right, size_t length);
```

```c
# src/memory/ft_memory_scan.c

# include "libft.h"

void *ft_memchr(const void *memory, int byte, size_t length) {
    const unsigned char *memory_byte;
    size_t index;

    memory_byte = memory;
    index = 0;
    
    while (index < length) {
        if (memory_byte[index] == (unsigned char)byte)
            return ((void *)memory_byte + index);
        ++index;
    }
    return (NULL);
}

int ft_memcmp(const void *left, const void *right, size_t length) {
    const unsigned char *left_byte;
    const unsigned char *right_byte;
    size_t index;

    left_byte = left;
    right_byte = right;
    index = 0;
    while (index < length) {
        if (left_byte[index] != right_byte[index])
            return ((int)left_byte[index] - (int)right_byte[index]);
        ++index;
    }
    return (0);
}
```

`ft_memchr`는 `const unsigned char *memory_byte`로 캐스팅한 뒤 `index`를 `0`부터 `length`까지 증가시키며 각 바이트가 `(unsigned char)byte`와 일치하는지 검사하고, 일치하면 해당 위치의 포인터를 `(void *)`로 캐스팅해 반환한다. 끝까지 못 찾으면 `NULL`을 반환하는 전형적인 선형 탐색 구조다.

`ft_memcmp`는 `left_byte`, `right_byte` 두 포인터를 `index`로 함께 순회하며 값이 다른 최초 지점을 찾으면 `(int)left_byte[index] - (int)right_byte[index]`를 반환하고, 끝까지 같으면 `0`을 반환한다. `unsigned char` 값을 `int`로 캐스팅한 뒤 뺄셈을 수행해 부호 있는 비교 결과를 만드는 방식으로, libc `memcmp`가 "차이의 부호"만 보장하는 관례를 따른 구현이다. 캐스팅 과정 없이 `unsigned char`끼리 그냥 뺄셈하면 언더 플로우가 발생할 수 있다.

### 문제 해결 방식
두 함수 모두 `length`로 검색/비교 범위를 명시적으로 제한하고 있어, 널 종료 문자열이 아닌 임의의 바이트 시퀀스(바이너리 데이터 등)에 대해서도 안전하게 동작하도록 구현했다. 별도의 엣지 케이스 방어 코드(예: `length`가 `0`인 경우의 조기 반환) 없이 `while` 루프 조건(`index < length`)이 자연스럽게 그 경우를 처리하는 구조다.