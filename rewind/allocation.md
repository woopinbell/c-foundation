# 동적 할당

## `feat(alloc): 0 초기화 메모리와 문자열 복제 추가`

### 구현
```c
# libft.h

void *ft_calloc(size_t count, size_t size);
char *ft_strdup(const char *text);
```

```c
# src/alloc/ft_allocate.c

#include "libft.h"
#include <stdlib.h>

void *ft_calloc(size_t count, size_t size) {
    size_t allocation_size;
    void *allocation;

    if (count != 0 && size > (size_t)-1 / count)
        return (NULL);
    allocation_size = count * size;
    if (allocation_size == 0)
        allocation_size = 1;
    allocation = malloc(allocation_size);
    if (allocation == NULL)
        return (NULL);
    ft_bzero(allocation, allocation_size);
    return (allocation);
}

char *ft_strdup(const char *text) {
    char *duplicate;
    size_t length;

    if (text == NULL)
        return (NULL);
    length = ft_strlen(text);
    if (length == (size_t)-1)
        return (NULL);
    duplicate = malloc(length + 1);
    if (duplicate == NULL)
        return (NULL);
    ft_memcpy(duplicate, text, length + 1);
    return (duplicate);
}
```

**`ft_calloc(count, size)`**는 표준 `calloc`을 재현한다.

- `malloc` + `ft_bzero`로 0 초기화된 메모리를 만드는 방식을 택했다. `size * count`를 미리 계산해 두는 게 아니라, 곱셈 전에 `count != 0 && size > (size_t)-1 / count` 조건으로 오버플로 여부를 먼저 검사한다. 이는 `count * size`가 `size_t` 범위를 넘어 랩어라운드되는 것을 막기 위한 전형적인 나눗셈 기반 오버플로 체크 패턴이다.

- `allocation_size`가 0이면(즉 `count`나 `size` 중 하나가 0이면) 1바이트를 할당하도록 보정한다. `malloc(0)`의 구현별 동작(NULL을 반환할 수도 있음)에 기대지 않고 항상 유효한 포인터를 돌려주려는 의도였다.

- 0 초기화는 직접 루프를 돌리지 않고 이미 만들어둔 `ft_bzero`를 재사용해서, 기존 memory 유틸 위에 쌓는 구조를 유지했다.

**`ft_strdup(text)`**는 `ft_strlen`으로 길이를 잰 뒤 `length + 1`(널 종단 포함)만큼 `malloc`하고 `ft_memcpy`로 복사한다.

- `text == NULL`일 때 바로 `NULL`을 반환해 방어한다.

- `length == (size_t)-1`일 때도 `NULL`을 반환하는 분기가 있는데, 이는 `ft_strlen`이 실패(혹은 특정 에러 상황)를 `(size_t)-1`로 표현한다는 것을 전제로 한 방어 코드다.

두 함수 모두 새로 만든 로직보다는 기존에 있던 `ft_bzero`, `ft_strlen`, `ft_memcpy` 위에 얇게 조립하는 방식으로 구현했다.

### 문제 해결 방식
1. **정수 오버플로 방지**: `count * size`를 먼저 계산하지 않고 나눗셈으로 오버플로 가능성을 미리 검사한 것은, 큰 `count`/`size` 조합이 들어왔을 때 실제 필요한 크기보다 작게 할당되어 버퍼 오버플로로 이어지는 전형적인 취약점을 막기 위한 것이다.

2. **0 크기 할당 처리**: `allocation_size == 0`일 때 1바이트로 보정하는 것도, `malloc(0)`의 플랫폼별 불확실한 동작에 의존하지 않으려는 방어적 선택이다.

`ft_strdup` 쪽은 NULL 입력 방어와 `ft_strlen`의 에러 반환값 방어가 있지만, 그 외의 특별한 동시성이나 성능 이슈를 다루고 있지는 않다.