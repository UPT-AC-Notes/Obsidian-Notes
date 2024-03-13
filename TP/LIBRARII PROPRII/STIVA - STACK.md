- structura de tip LIFO -> last in first out
- reprezentata ca un vector cu un index la ultimul element din acesta
### OPERATII

- INIT -> initializare
- PUSH -> adaugare element (la varful stivei)
- POP -> stergerea unui element (de la varful stivei)
- PEEK -> vizualizarea elementului din varf

## TIPUL DE DATA

- poate fi folosit orice tip de data intr-o stiva
- notat cu un typedef

- top -> index la varful stivei
- capacity -> marimea maxima a stivei (poate fi modificata cu alocare dinamica)
- pointer la data -> vectorul de date din stiva

```c
typedef int stack_data;

typedef struct {
  size_t top;
  size_t capacity;
  stack_data *data;
} stack;
```

## INITIALIZARE

- setam capacitatea initiala
- alocam dinamic vectorul de date
- daca esueaza alocarea, capacitatea va deveni 0
- daca nu, returnam stiva

```c
stack init_stack(size_t capacity) {

  stack st = (stack){0, capacity, NULL};

  st.data = (stack_data *)malloc(capacity * sizeof(stack_data));

  if (st.data == NULL) {
    if (STACK_DEBUG)
      printf("memory allocation failed\n");
    st.capacity = 0;
  }

  if (STACK_DEBUG) {
    printf("initialization successful\n");
    printf("stack capacity set to: %zu\n", st.capacity);
  }

  return st;
}
```

## EMPTY STACK CHECK

- daca varful stivei este setat la 0, atunci stiva este goala

```c
int stack_is_empty(stack *st) {
  if (STACK_DEBUG && st->top == 0)
    printf("stack empty\n");
  return st->top == 0;
}
```

## FULL STACK CHECK

- daca varful stivei este egal (sau mai mare) decat capacitatea, atunci stiva este plina

```c
int stack_is_full(stack *st) {
  if (STACK_DEBUG && st->top >= st->capacity)
    printf("stack full\n");
  return st->top >= st->capacity;
}
```

## STACK REALLOC

- daca realocarea dinamica este selectata, capacitatea stivei poate fi crescuta cu o valoare data

```c
int stack_realloc(stack *st) {
  stack_data *temp = (stack_data *)realloc(
      st->data, (st->capacity + STACK_CHUNK) * sizeof(stack_data));

  if (temp == NULL) {
    if (STACK_DEBUG) {
      printf("memory reallocation failed\n");
      printf("stack capacity remains at: %zu\n", st->capacity);
      printf("no push happened\n");
    }
    return 0;
  }

  st->data = temp;
  st->capacity += STACK_CHUNK;

  if (STACK_DEBUG)
    printf("stack capacity increased to: %zu\n", st->capacity);

  return 1;
}
```

## PUSH

- daca stiva este plina si modul dinamic nu este selectat, nu se va face nimic
- daca stiva este plina si modul dinamic este selectat, se va face realocare iar in caz de eroare de memorie va returna -1
- daca stiva nu este plina, noul element va fi pus in varful stivei, top-ul va creste

```c
int push(stack *st, stack_data data) {

  // full -> dynamic -> realloc -> fail or not
  // full -> static -> no push

  if (stack_is_full(st)) {
    if (STACK_DYNAMIC) {
      if (!stack_realloc(st))
        return -1;
    } else {
      if (STACK_DEBUG)
        printf("no push happened\n");
      return 0;
    }
  }
  if (STACK_DEBUG)
    printf("push %d\n", data);

  st->data[st->top++] = data;
  return 1;
}
```

## POP

- daca stiva este goala, nu se va efectua nicio operatie
- daca stiva nu este goala, varful stivei va scadea cu 1

```c
int pop(stack *st) {
  if (stack_is_empty(st)) {
    return 0;
  }

  if (STACK_DEBUG)
    printf("pop %d\n", st->data[st->top - 1]);

  st->top--;
  return 1;
}
```

## PEEK

- daca stiva este goala, returneza 0
- daca stiva nu este goala, returneaza elementul din varful stivei

```c
stack_data peek(stack *st) {
  if (stack_is_empty(st)) {
    return 0;
  }

  if (STACK_DEBUG)
    printf("peek %d\n", st->data[st->top - 1]);

  return st->data[st->top - 1];
}
```

## PRINT

```c
void print_stack(stack *st) {
  for (size_t i = 0; i < st->top; i++) {
    printf("%d ", st->data[i]);
  }
}
```

## FREE

- elibereaza datele din stiva si o initializeaza la loc

```c
void free_stack(stack *st) {

  if (STACK_DEBUG)
    printf("free stack\n");

  free(st->data);
  *st = (stack){0, 0, NULL};
}
```