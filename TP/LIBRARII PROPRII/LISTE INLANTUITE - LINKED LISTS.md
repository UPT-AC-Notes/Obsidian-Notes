- structura de tip LIFO -> last in first out
- reprezentata ca un vector cu un index la ultimul element din acesta
### OPERATII

- INIT -> initializare
- PUSH -> adaugare element (la varful stivei)
- POP -> stergerea unui element (de la varful stivei)
- PEEK -> vizualizarea elementului din varf

## TIPUL DE DATA

- poate fi folosit orice tip de data intr-o lista
- notat cu un typedef

- data -> tipul de data continut de nod
- pointer la un struct nod, o structura definita recursiv
- pointer la nodul cap si la nodul de final

```c
typedef int list_data_t;

typedef struct node {
  list_data_t data;
  struct node *next;
} node_t;

typedef struct {
  node_t *head;
  node_t *rear;
} list_t;
```

## INITIALIZARE

```c
list_t init_list(void) { return (list_t){NULL, NULL}; }
```

## ADD FRONT NODE

- cream un nou nod cu datele din parametru
- nodul va avea ca succesor pe capul listei
- setam capul listei sa fie acel nou nod
- daca lista este goala, setam si coada listei sa fie tot acel nod

```c
int add_front_node(list_t *list, list_data_t input_data) {

  node_t *new_node = (node_t *)malloc(sizeof(node_t));

  if (new_node == NULL) {
    if (LIST_DEBUG)
      printf("memory allocation failed\n");

    return 0;
  }

  if (LIST_DEBUG)
    printf("new node linked to front / data linked: %d\n", input_data);

  *new_node = (node_t){input_data, list->head};

  list->head = new_node;

  if (list->rear == NULL) {
    list->rear = new_node;
  }

  return 1;
}
```

## FULL STACK CHECK

- cream un nou nod cu datele din parametru
- coada listei va avea ca succesor noul nod 
- setam coada listei sa fie acel nou nod
- daca lista este goala, adaugam doar un nod nou

```c
int add_rear_node(list_t *list, list_data_t input_data) {

  if (list->rear == NULL) {
    if (!add_front_node(list, input_data)) {
      return 0;
    }
    return 1;
  }

  node_t *new_node = (node_t *)malloc(sizeof(node_t));

  if (new_node == NULL) {
    if (LIST_DEBUG)
      printf("memory allocation failed\n");

    return 0;
  }

  if (LIST_DEBUG)
    printf("new node linked to rear / data linked: %d\n", input_data);

  *new_node = (node_t){input_data, NULL};

  list->rear->next = new_node;
  list->rear = list->rear->next;

  return 1;
}
```

## STACK REALLOC

- daca lista este goala, nu se face nicio actiune
- daca lista contine doar un element, acesta este eliberat si lista este reinitializata
- daca lista mai contine elemente, asignam un nod temporar pentru capul listei, capul listei devine urmatorul nod dupa cap, eliberam nodul temporar

```c
int pop_front(list_t *list) {
  if (list->head == NULL) {
    printf("list is empty, no action taken\n");
    return -1;
  }

  if (list->head->next == NULL) {

    if (LIST_DEBUG)
      printf("removed last item in list\n");

    free(list->head);

    *list = (list_t){NULL, NULL};
    
    return 0;
  }

  if (LIST_DEBUG)
    printf("pop front / data: %d\n", list->head->data);

  node_t *temp = list->head;
  list->head = list->head->next;
  free(temp);

  return 1;
}
```

## PUSH

- daca lista este goala, nu se face nicio actiune
- daca lista contine doar un element, acesta este eliberat si lista este reinitializata
- daca lista mai contine elemente, iteram prin elementele listei cu un nod temporar, pana cand intalnim nodul care il are ca next pe nodul de coada, eliberam nodul urmator al nodului temporar, il setam pe NULL iar asignam coada ca sa fie nodul temporar

```c
int pop_rear(list_t *list) {

  if (list->head == NULL) {
    printf("list is empty, no action taken\n");
    return -1;
  }

  if (list->head->next == NULL) {

    if (LIST_DEBUG)
      printf("removed last item in list\n");

    free(list->head);
    *list = (list_t){NULL, NULL};

    return 0;
  }

  if (LIST_DEBUG)
    printf("pop front / data: %d\n", list->rear->data);

  node_t *iterator = list->head;

  while (iterator->next != list->rear) {
    iterator = iterator->next;
  }

  free(iterator->next);
  iterator->next = NULL;

  list->rear = iterator;

  return 1;
}
```

## PRINT LIST

- definim un nod temporar pentru iterare prin lista
- cat timp nodul nu este unul NULL, printam datele din nod, nodul devine nodul urmator al acestuia

```c
void print_list(list_t *list) {

  node_t *current = list->head;

  while (current != NULL) {
    printf("%d -> ", current->data);
    current = current->next;
  }

  printf("NULL\n");
}
```

## FREE LIST

- definim un nod pentru iterare iar unul temporar pentru eliberare
- nodul de iterare incepe la capul listei, se termina cand devine NULL
- nodul temporar devine nodul de iterare, nodul de iterare devine urmatorul nod al sau, eliberam nodul temporar

```c
void free_list(list_t *list) {
  node_t *current = list->head;

  while (current != NULL) {
    node_t *temp = current;
    current = current->next;
    free(temp);
  }

  *list = (list_t){NULL, NULL};
}
```