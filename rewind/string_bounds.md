# 문자열 기본 함수 (길이/복사/검색)

## 1. `feat(string): 문자열 길이 계산과 제한 복사/붙이기 추가`

### 구현
```c
# libft.h

size_t ft_strlen(const char *text);
size_t ft_strlcpy(char *dest, const char *src size_t capacity);
size_t ft_strlcat(char *dest, const char *src size_t capacity);
```

```c
# src/string/ft_string_bounds.c

# include "libft.h"

size_t ft_strlen(const char *text) {
    size_t length;
    lenght = 0;
    while (text[length] != '\0') {
        ++length;
    }
    return (length);
}

size_t ft_strlcpy(char *dest, const char *src, size_t capacity) {
    size_t src_length;
    size_t idx;

    src_length = ft_strlen(src);
    if (capacity == 0)
        return src_length;
    idx = 0;
    while (src[idx] != '\0' && idx + 1 < capacity) {
        dest[idx] = src[idx];
        ++idx;
    }
    dest[idx] = '\0';
    return src_length;
}

size_t ft_strlcat(char *dest, const char *src, size_t capacity) {
    size_t dest_length;
    size_t src_length;
    size_t idx;

    dest_length = 0;
    while (dest_length < capacity && dest[dest_length] != '\0')
        ++dest_lenght;
    src_length = ft_strlen(src);
    if (dest_length == capacity)
        return capacity + src_length;
    idx = 0;
    while (src[idx] != '\0' && dest_length + idx + 1 < capacity) {
        dest[dest_length + idx] = src[idx];
        ++idx;
    }
    dest[dest_length + idx] = '\0';
    return (dest_length + src_length);
}
```

`ft_strlen`은 널 종료 문자를 만날 때까지 인덱스를 증가시키는 가장 단순한 형태다. `ft_strlcpy`와 `ft_strlcat`은 표준 `strlcpy`/`strlcat` 시그니처(`capacity`를 받는 버전)를 그대로 따르는데, 두 함수 모두 반환값을 "실제로 복사한 길이"가 아니라 "capacity 제약이 없었다면 필요했을 전체 길이"로 설계했다. `ft_strlcpy`는 `src_length`를 먼저 `ft_strlen`으로 구해두고 반환하며, `ft_strlcat`은 기존 `dest`의 길이를 `capacity`를 넘지 않는 범위에서 직접 세어 `dest_length`로 삼은 뒤 그 뒤에 이어붙인다.

`ft_strlcat`의 특이점은 `if (dest_length == capacity) return (capacity + src_length);` 부분이다. 이는 `dest`가 이미 널 종료 없이 capacity를 꽉 채운 비정상 상태를 가정한 방어 코드다. 이 경우 어디까지가 문자열인지 알 수 없으므로 아무것도 이어붙이지 않고 `capacity + src_length`를 반환해 "실패했음"을 값으로 알리는 방식이다.

### 문제 해결 방식
`capacity == 0`일 때 `ft_strlcpy`가 `dest`에 아예 손대지 않고 `src_length`만 반환하는 분기, 그리고 두 함수 모두 루프 조건에 `idx + 1 < capacity`처럼 널 종료 문자 한 칸을 항상 남겨두는 방식이다. 이는 "capacity를 넘겨받은 버퍼 크기 그대로 쓰다가 널 종료를 빠뜨리는" 오버플로우 버그를 원천 차단하려는 설계다.

## 2. `feat(string): 문자의 첫/마지막 위치 검색을 추가`

### 구현
```c
# libft.h

char	*ft_strchr(const char *text, int c);
char	*ft_strrchr(const char *text, int c);
```

```c
# src/string/ft_string_search.c

#include "libft.h"

char	*ft_strchr(const char *text, int c) {
    unsigned char	target;
    size_t idx;

    target = (unsigned char)c;
    idx = 0;
    while (1) {
        if ((unsigned char)text[idx] == target)
            return ((char *)(text + idx));
        if (text[idx] == '\0')
            return (NULL);
        ++idx;
    }
}

char	*ft_strrchr(const char *text, int c) {
    unsigned char	target;
    char *match;

    target = (unsigned char)c;
    match = NULL;
    while (1) {
        if ((unsigned char)*text == target)
            match = (char *)text;
        if (*text == '\0')
            return (match);
        ++text;
    }
}
```

`ft_string_search.c` 파일을 새로 만들며 이전 커밋의 `ft_string_bounds.c`와 역할을 분리했다. `ft_strlen` 등 앞서 구현한 함수를 직접 호출하지는 않고, 두 함수 모두 자체적으로 루프를 도는 형태로 독립적으로 구현했다.

`ft_strchr`은 `while (1)` 무한 루프 안에서 먼저 현재 문자가 타겟과 일치하는지 검사하고, 그 다음에 널 종료 문자인지 검사하는 순서를 취한다. 이 순서 덕분에 `c`로 `'\0'`이 들어와도(표준 `strchr`이 널 종료 위치를 찾아주는 동작과 동일하게) 정상적으로 그 위치를 찾아 반환한다. `ft_strrchr`도 같은 구조를 쓰되, 일치할 때마다 `match`를 계속 갱신해 두었다가 널 종료 지점에서 마지막으로 갱신된 `match`를 반환하는 방식으로 "마지막 위치"를 구현했다.

두 함수 모두 `c`를 `unsigned char target`으로 먼저 캐스팅하고 비교도 `(unsigned char)text[idx]`로 맞추었다.

### 문제 해결 방식
`int c` 인자를 `char`가 아니라 `unsigned char`로 캐스팅해서 비교하는 부분은, 플랫폼에 따라 `char`가 `signed`일 수 있어 발생하는 부호 확장 문제(예: `0x80` 이상의 문자를 비교할 때 음수로 해석되는 문제)를 막기 위한 처리이다.

## 3. `feat(string): 범위 비교와 부분 문자열 검색을 추가`

### 구현
```c
# libft.h

int		ft_strncmp(const char *left, const char *right, size_t length);
char	*ft_strnstr(const char *haystack, const char *needle, size_t length);
```

```c
# src/string/ft_string_search.c

int	ft_strncmp(const char *left, const char *right, size_t length) {
    size_t	idx;

    idx = 0;
    while (idx < length) {
        if ((unsigned char)left[idx] != (unsigned char)right[idx])
            return ((int)(unsigned char)left[idx] - (int)(unsigned char)right[idx]);
        if (left[idx] == '\0')
            return (0);
        ++idx;
    }
    return (0);
}

char	*ft_strnstr(const char *haystack, const char *needle, size_t length) {
    size_t	haystack_idx;
    size_t	needle_idx;

    if (*needle == '\0')
        return ((char *)haystack);
    haystack_idx = 0;
    while (haystack_idx < length && haystack[haystack_idx] != '\0') {
        needle_idx = 0;
        while (needle[needle_idx] != '\0'
              && needle_idx < length - haystack_idx
              && haystack[haystack_idx + needle_idx]
              == needle[needle_idx])
            ++needle_idx;
        if (needle[needle_idx] == '\0')
            return ((char *)(haystack + haystack_idx));
        ++haystack_idx;
    }
    return (NULL);
}
```

`ft_strncmp`는 `length`만큼 인덱스를 돌며 문자가 다르면 그 자리에서 `(unsigned char)` 차이값을 반환하고, 같은 도중에 널 종료를 만나면 그 지점에서 동일한 문자열로 보고 `0`을 반환한다. `ft_strnstr`은 이중 루프 구조로, 바깥 루프가 `haystack`의 각 시작 위치를 훑고 안쪽 루프가 그 위치에서 `needle`과 얼마나 일치하는지를 센다. 안쪽 루프 조건에 `needle_idx < length - haystack_idx`를 넣어, `haystack_idx`가 커질수록 비교 가능한 남은 길이가 줄어드는 것을 반영했다.

`if (*needle == '\0') return ((char *)haystack);`로 빈 문자열 검색을 첫 줄에서 바로 처리했다.

### 문제 해결 방식
`ft_strnstr`에서 `length`라는 상한을 두 군데(바깥 루프의 `haystack_idx < length`, 안쪽 루프의 `needle_idx < length - haystack_idx`)에 반영한 것은, `haystack`이 `length`를 넘어서까지 이어지는 메모리라도 그 경계를 넘어 읽지 않도록 하려는 의도였다. 다만 `length - haystack_idx` 계산은 `haystack_idx`가 `length`보다 클 수 없다는 전제하에서만 안전한데, 바깥 루프 조건이 이를 보장하고 있어 언더플로우에 대한 위험이 없다.