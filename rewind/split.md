# 문자열 분리 (split)

## `feat(string): 실패 시 정리되는 문자열 분리 구현`

### 구현

```c
# libft.h

char **ft_split(const char *text, char delimiter);
```

```c
# src/string/ft_split.c

#include "libft.h"
#include <stdlib.h>

static size_t	count_fields(const char *text, char delimiter) {
	size_t	count;
	size_t	index;

	count = 0;
	index = 0;
	while (text[index] != '\0') {
		  while (text[index] == delimiter && text[index] != '\0')
			    index++;
		  if (text[index] != '\0')
			    count++;
		  while (text[index] != delimiter && text[index] != '\0')
			    index++;
	}
	return (count);
}

static char	*copy_field(const char *start, size_t length) {
	char	*field;

	field = malloc(length + 1);
	if (field == NULL)
		  return (NULL);
	ft_memcpy(field, start, length);
	field[length] = '\0';
	return (field);
}

static void	free_fields(char **fields, size_t count) {
    while (count > 0) {
        count--;
        free(fields[count]);
    }
    free(fields);
}

char	**ft_split(const char *text, char delimiter) {
    char	**fields;
    size_t	text_index;
    size_t	field_index;
    size_t	start;

    if (text == NULL)
        return (NULL);
    fields = ft_calloc(count_fields(text, delimiter) + 1, sizeof(char *));
    if (fields == NULL)
        return (NULL);
    text_index = 0;
    field_index = 0;
    while (text[text_index] != '\0') {
        while (text[text_index] == delimiter && text[text_index] != '\0')
            text_index++;
        start = text_index;
        while (text[text_index] != delimiter && text[text_index] != '\0')
            text_index++;
        if (text_index > start) {
            fields[field_index] = copy_field(text + start, text_index - start);
            if (fields[field_index] == NULL)
                return (free_fields(fields, field_index), NULL);
            field_index++;
        }
    }
    return (fields);
}
```

`ft_split`은 문자열을 특정 구분자(`delimiter`) 기준으로 나눠 문자열 배열(`char **`)로 반환하는 함수로, 내부적으로 세 개의 정적 헬퍼 함수(`count_fields`, `copy_field`, `free_fields`)와 하나의 공개 함수로 구성된다.

핵심 구조는 "먼저 몇 개로 나뉠지 센 다음, 그 개수만큼 배열을 할당하고, 다시 순회하며 채운다"는 2-패스(two-pass) 방식이다.

- `count_fields`: 구분자를 건너뛰고, 구분자가 아닌 구간(필드)을 하나씩 세는 방식으로 총 필드 개수를 계산한다. `while (text[index] == delimiter ...)`로 구분자를 스킵한 뒤, 문자열 끝이 아니면 카운트를 올리고, 다시 `while (text[index] != delimiter ...)`로 필드 구간을 건너뛰는 패턴을 반복한다.
- `ft_split` 본문에서는 `count_fields`로 얻은 개수 + 1(NULL 종료용)만큼 `ft_calloc`으로 배열을 할당한다. `calloc`을 쓴 것은 각 슬롯이 기본적으로 `NULL`로 초기화되어야 하기 때문으로 보이며, 이는 뒤에서 설명할 실패 처리 로직과 맞물린다.
- 이후 다시 한 번 문자열을 순회하면서, 구분자를 스킵하고(`start` 위치 기록) 필드 구간의 끝을 찾은 뒤 `copy_field(text + start, text_index - start)`로 해당 구간만 복사해 배열에 채워 넣는다.
- `copy_field`는 `malloc(length + 1)` 후 `ft_memcpy`로 복사하고 널 종단시키는, 표준적인 부분 문자열 복사 로직이다.

### 문제 해결 방식
주요 문제는 **메모리 할당 실패 시의 정리(cleanup)** 다.

- `text == NULL`이면 바로 `NULL`을 반환해 잘못된 입력에 대한 방어를 한다.
- `fields = ft_calloc(...)`이 실패하면 그대로 `NULL`을 반환한다 (아직 아무 필드도 복사하지 않았으므로 추가 정리가 필요 없는 상태).
- 반면 필드를 복사하는 도중(`copy_field`) 실패가 나면, 이미 할당해 둔 이전 필드들을 그냥 버리지 않고 `free_fields(fields, field_index)`로 **지금까지 채운 만큼만** 정리한 뒤 `NULL`을 반환한다. `field_index`를 인자로 넘겨 "몇 개까지 채워졌는지"를 정확히 추적하여 아직 채워지지 않은 뒤쪽의 `NULL` 슬롯까지 잘못 해제하지 않도록 범위를 맞춘다.
- `free_fields`는 `count`를 뒤에서부터 감소시키며 각 필드를 `free`하고 마지막에 배열 자체도 `free`하는, 부분 실패 상황을 위한 전용 해제 함수다.