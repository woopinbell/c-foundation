# 연결 리스트

## `feat(list): 연결 리스트 노드 생성과 앞 삽입을 구현`

```c
# libft.h
```
```c
# src/list/ft_list_basic.c

#include "libft.h"
#include <stdlib.h>

t_list	*ft_lstnew(void *content) {
	  t_list	*node;

	  node = malloc(sizeof(t_list));
    if (node == NULL)
        return (NULL);
    node->content = content;
    node->next = NULL;
    return (node);
}

void	ft_lstadd_front(t_list **list, t_list *node) {
    if (list == NULL || node == NULL)
        return ;
    node->next = *list;
    *list = node;
}
```

가장 먼저 `libft.h`에 `t_list` 구조체를 정의하는 것부터 시작한다. 이후 모든 리스트 함수의 시그니처가 이 타입에 의존하므로 기반 작업에 해당한다.

`t_list`는 `content`(void*)와 `next`(자기 참조 포인터)만 가진 단방향 연결 리스트 노드다. `ft_lstnew`는 `malloc`으로 노드를 할당하고 `content`를 채운 뒤 `next`를 `NULL`로 초기화해서 반환한다. `ft_lstadd_front`는 새 노드의 `next`를 현재 리스트 head로 연결한 뒤 `*list`를 갱신하는 전형적인 단일 연결 리스트 push-front 패턴이다.

### 문제 해결 방식
`ft_lstnew`는 `malloc` 실패 시 `NULL`을 그대로 반환해 호출자가 실패를 감지할 수 있게 한다. `ft_lstadd_front`는 `list == NULL || node == NULL`을 먼저 검사해 널 포인터 역참조를 방지한다.

## `feat(list): 연결 리스트 크기와 마지막 노드를 조회`

```c
# libft.h

int ft_lstsize(t_lit *list);
t_list *ft_lstlast(t_list *list)
```
```c
# src/list/ft_list_basic.c

int	ft_lstsize(t_list *list) {
    int	size;

    size = 0;
    while (list != NULL) {
        size++;
        list = list->next;
    }
    return (size);
}

t_list	*ft_lstlast(t_list *list) {
    if (list == NULL)
        return (NULL);
    while (list->next != NULL)
        list = list->next;
    return (list);
}
```

앞서 만든 `t_list`와 `ft_lstnew`/`ft_lstadd_front` 위에 조회 기능을 추가하는 단계다. 아직 뒤 삽입 함수는 없고, 이번 커밋에서 만든 `ft_lstlast`가 다음 구현의 재료가 된다.

두 함수 모두 `list`를 순회하는 `while (list != NULL)` 루프를 쓴다는 점에서 동일한 패턴을 공유한다. `ft_lstsize`는 순회하면서 카운터를 증가시키고, `ft_lstlast`는 `list->next != NULL`을 조건으로 삼아 마지막 노드에서 멈춘다(즉 `list` 자체가 아니라 `list->next`를 검사해 널을 가리키기 직전에 멈추는 방식).

### 문제 해결 방식
`ft_lstlast`는 빈 리스트(`list == NULL`) 케이스를 먼저 걸러 `NULL`을 반환하는 방어 코드를 두었다. `ft_lstsize`도 `list == NULL`이면 루프에 진입하지 않아 자연스럽게 0을 반환한다.

## `feat(list): 연결 리스트 뒤 삽입을 구현`

```c
# libft.h

void ft_lstadd_back(t_list **list, t_list *node);
```
```c
# src/list/ft_list_basic.c

void	ft_lstadd_back(t_list **list, t_list *node) {
    t_list	*last;

    if (list == NULL || node == NULL)
        return ;
    if (*list == NULL) {
        *list = node;
        return ;
    }
    last = ft_lstlast(*list);
    last->next = node;
}
```

앞서 만든 `ft_lstlast`를 그대로 재사용해서 구현한다. 새 로직을 처음부터 짜지 않고 이미 있는 탐색 함수 위에 삽입 로직만 얹은 형태다.

`ft_lstadd_back`은 먼저 빈 리스트인지(`*list == NULL`) 확인해 그 경우 `node`를 그대로 head로 설정하고 종료한다. 비어 있지 않으면 `ft_lstlast(*list)`로 마지막 노드를 찾아 그 뒤에 `node`를 연결한다. `last->next = node`만 하고 `node->next`는 별도로 건드리지 않는데, 이는 `ft_lstnew`가 이미 `next`를 `NULL`로 초기화해 둔 것을 전제로 한다.

### 문제 해결 방식
빈 리스트에 대한 뒤 삽입을 별도 분기로 처리한 것이 핵심이다. 이 분기가 없으면 `ft_lstlast(NULL)`이 `NULL`을 반환하고 `last->next`에서 널 역참조가 발생하므로, 이 커밋은 그 엣지 케이스를 방어한다.

## `feat(list): 연결 리스트 순회와 삭제 구현`

```c
# libft.h

void ft_lstdelone(t_list *node, void (*del)(void *));
void ft_lstclear(t_list **list, void (*del)(void *));
void ft_lstiter(t_list *list, void (*function)(void *));
```
```c
# src/list/ft_list_lifecycle.c

#include "libft.h"
#include <stdlib.h>

void	ft_lstdelone(t_list *node, void (*del)(void *)) {
	  if (node == NULL || del == NULL)
		    return ;
    del(node->content);
    free(node);
}

void	ft_lstclear(t_list **list, void (*del)(void *)) {
    t_list	*next;

    if (list == NULL || del == NULL)
        return ;
    while (*list != NULL) {
        next = (*list)->next;
        ft_lstdelone(*list, del);
        *list = next;
    }
}

void	ft_lstiter(t_list *list, void (*function)(void *)) {
    if (function == NULL)
        return ;
    while (list != NULL) {
        function(list->content);
        list = list->next;
    }
}
```

별도 파일(`ft_list_lifecycle.c`)로 분리해 리스트의 소멸 관련 함수(`ft_lstdelone`, `ft_lstclear`)와 순회 함수(`ft_lstiter`)를 함께 추가한다.

`ft_lstdelone`은 `del` 콜백으로 `content`를 해제한 뒤 노드 자체를 `free`하는, 단일 노드 단위의 해제 함수다. `ft_lstclear`는 `next`를 미리 저장해 둔 뒤 `ft_lstdelone`을 호출하고 `*list`를 다음 노드로 옮기는 식으로 전체 리스트를 순회하며 해제한다. `ft_lstiter`는 해제 없이 `function` 콜백만 각 노드의 `content`에 적용하는 순수 순회 함수다.

### 문제 해결 방식
`ft_lstclear`에서 `next = (*list)->next`를 `ft_lstdelone` 호출 전에 미리 저장하는 부분이 핵심 방어 코드다. 노드를 해제한 뒤에 `next`를 읽으면 이미 해제된 메모리에 접근(use-after-free)하게 되므로, 순서를 먼저 저장 → 해제 → 이동으로 고정해 이를 막는다. 세 함수 모두 콜백 포인터(`del`, `function`)가 `NULL`인 경우를 검사해 잘못된 함수 포인터 호출을 방지한다.

## `feat(list): 실패 시 정리되는 리스트 변환 구현`

```c
# libft.h

t_list *ft_lstmap(t_list *list, void *(*function)(void *), void (*del)(void *));
```
```c
# src/list/ft_list.c

#include "libft.h"

t_list *ft_lstmap(t_list *list, void *(*function)(void *), void (*del)(void *)) {
    t_list	*mapped;
    t_list	*tail;
    t_list	*node;
    void	*mapped_content;

    if (function == NULL || del == NULL)
        return (NULL);
    mapped = NULL;
    tail = NULL;
    while (list != NULL) {
        mapped_content = function(list->content);
        node = ft_lstnew(mapped_content);
        if (node == NULL) {
            del(mapped_content);
            ft_lstclear(&mapped, del);
            return (NULL);
        }
        if (mapped == NULL)
            mapped = node;
        else
            tail->next = node;
        tail = node;
        list = list->next;
    }
    return (mapped);
}
```

지금까지 만든 생성/해제 기능이 모두 재료로 쓰이는, 이 시리즈의 마지막 조립 단계에 해당한다.

`ft_lstmap`은 원본 `list`를 순회하며 각 노드의 `content`에 `function`을 적용한 결과(`mapped_content`)로 새 노드를 만들어 새로운 리스트(`mapped`)를 구성한다. `tail` 포인터를 별도로 유지하면서 `mapped == NULL`이면 head로 설정하고, 아니면 `tail->next`에 이어 붙이는 방식으로 뒤 삽입을 반복해 매번 `ft_lstlast`로 끝까지 탐색하는 비용 없이 O(n)에 새 리스트를 만든다.

### 문제 해결 방식
`function(list->content)` 호출 이후 그 결과로 `ft_lstnew`를 호출했는데 이 `malloc`이 실패하는 경우를 명시적으로 처리한다. 이때 이미 생성된 `mapped_content`를 `del`로 해제하고, 그때까지 만들어 둔 `mapped` 리스트 전체도 `ft_lstclear(&mapped, del)`로 정리한 뒤 `NULL`을 반환한다. 즉 중간에 실패하더라도 절반만 만들어진 리스트가 메모리에 남지 않도록 롤백하는 것이 이 커밋의 핵심 문제 해결 지점이다. 또한 `function == NULL || del == NULL` 검사로 필수 콜백이 없는 호출 자체를 막는다.
