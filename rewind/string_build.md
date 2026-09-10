# 문자열 생성/조합 작업 해설서

## 구현 흐름 개요

## 1. `feat(string): 부분 문자열 생성을 구현`

### 구현
```c
# libft.h

char *ft_substr(const char *text, unsigned int start, size_t length);
```
```c
# src/string/ft_string_build.c

#include "libft.h"
#include <stdlib.h>

char	*ft_substr(const char *text, unsigned int start, size_t length) {
    char	*sub_str;
    size_t	text_length;

    if (text == NULL)
        return (NULL);
    text_length = ft_strlen(text);
    if ((size_t)start >= text_length)
        return (ft_strdup(""));
    if (length > text_length - (size_t)start)
        length = text_length - (size_t)start;
    sub_str = malloc(length + 1);
    if (sub_str == NULL)
        return (NULL);
    ft_memcpy(sub_str, text + start, length);
    sub_str[length] = '\0';
    return (sub_str);
}
```

`ft_substr`은 `text + start`부터 `length`만큼(또는 문자열 끝까지 중 더 짧은 쪽) 잘라 새로 `malloc`한 버퍼에 복사해 반환한다. `start`가 문자열 길이보다 크거나 같으면 빈 문자열을 반환하고, `length`가 남은 길이보다 크면 남은 길이로 잘라낸다. 

### 문제 해결 방식

`start`가 문자열 범위를 벗어난 경우와 요청한 `length`가 실제 남은 길이보다 긴 경우를 처리하여 `ft_memcpy`가 문자열 끝을 넘어 읽는 버퍼 오버리드가 되는 것을 방지했다. 또한 `text == NULL`일 때 바로 `NULL`을 반환해 널 포인터 역참조를 막았다.

## 2. `feat(string): 문자열 결합을 구현`

### 구현

```c
# libft.h

char	*ft_strjoin(const char *left, const char *right);
```
```c
# src/string/ft_string_build.c

char	*ft_strjoin(const char *left, const char *right) {
    char	*joined;
    size_t	left_length;
    size_t	right_length;

    if (left == NULL || right == NULL)
        return (NULL);
    left_length = ft_strlen(left);
    right_length = ft_strlen(right);
    if (right_length == (size_t)-1 ||
        left_length > (size_t)-2 - right_length)
        return (NULL);
    joined = malloc(left_length + right_length + 1);
    if (joined == NULL)
        return (NULL);
    ft_memcpy(joined, left, left_length);
    ft_memcpy(joined + left_length, right, right_length + 1);
    return (joined);
}
```

`left`와 `right` 두 문자열을 이어 붙인 새 문자열을 반환한다. `ft_memcpy(joined, left, left_length)`로 앞부분을 복사하고, `ft_memcpy(joined + left_length, right, right_length + 1)`로 뒷부분과 널 종료 문자까지 한 번에 복사해 별도로 `joined[...] = '\0'`을 쓰지 않는 방식으로 구현했다.

### 문제 해결 방식
`left_length + right_length + 1`을 그대로 `malloc`에 넘기면 두 길이의 합이 `size_t` 최댓값을 넘을 때 정수 오버플로가 발생해 실제 필요한 크기보다 작은 버퍼가 할당될 수 있으며 그 상황을 `left_length > (size_t)-2 - right_length` 조건으로 미리 걸러낸다. `(size_t)-2`는 `SIZE_MAX - 1`이므로, `+1`(널 종료 문자)까지 포함한 합이 오버플로하지 않는지를 사전에 확인한다.

## 3. `feat(string): 양끝 문자 집합 제거를 구현`

### 구현

```c
# libft.h

char	*ft_strtrim(const char *text, const char *set);
```
```c
# src/string/ft_string_build.c

static int	is_in_set(char character, const char *set) {
    while (*set != '\0') 	{
        if (*set == character)
            return (1);
        set++;
    }
    return (0);
}

char	*ft_strtrim(const char *text, const char *set) {
    char	*trimmed;
    size_t	start;
    size_t	end;

    if (text == NULL || set == NULL)
        return (NULL);
    start = 0;
    while (text[start] != '\0' && is_in_set(text[start], set))
        start++;
    end = ft_strlen(text);
    while (end > start && is_in_set(text[end - 1], set))
        end--;
    trimmed = malloc(end - start + 1);
    if (trimmed == NULL)
        return (NULL);
    ft_memcpy(trimmed, text + start, end - start);
    trimmed[end - start] = '\0';
    return (trimmed);
}
```

`is_in_set`은 `set` 문자열을 순회하며 `character`가 그 안에 포함되는지 확인하는 단순 선형 탐색 함수다. `ft_strtrim`은 이를 이용해 앞쪽(`start`)에서 `set`에 속한 문자를 건너뛰고, 뒤쪽(`end`)에서도 `set`에 속한 문자를 건너뛴 뒤, 그 사이 구간만 새 버퍼에 복사한다.


### 문제 해결 방식
뒤쪽 트리밍 루프에서 `end > start` 조건을 함께 검사하여 문자열 전체가 `set`에 속한 문자로만 이루어진 경우(예: `text`가 전부 공백) `end`가 `start`보다 작아지도록 계속 감소해 `end - start`가 음수(실제로는 `size_t`이므로 거대한 양수)가 되는 언더플로가 되는 것을 막는다. 이 덕분에 최악의 경우에도 `end == start`인 빈 문자열이 정상적으로 반환된다.