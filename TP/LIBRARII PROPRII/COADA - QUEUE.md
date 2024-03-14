- structura de tip FIFO -> first in first out
- reprezentata ca un vector cu un index la primul si ultimul element din acesta
### OPERATII

- INIT -> initializare
- ENQUEUE -> adaugare element (la varful stivei)
- DEQUEUE -> stergerea unui element (de la varful stivei)
- PEEK_HEAD -> vizualizarea elementului din capul cozii
- PEEK_TAIL -> vizualizarea elementului la finalul cozii

## TIPUL DE DATA

- poate fi folosit orice tip de data intr-o coada
- notat cu un typedef

- head -> index la capul cozii
- tail -> index la finalul cozii
- capacity -> marimea maxima a cozii (poate fi modificata cu alocare dinamica)
- pointer la data -> vectorul de date din stiva

```c
typedef int queue_data;

typedef struct queue {
  size_t head;
  size_t tail;
  size_t capacity;
  queue_data *data;
} queue;
```

## INITIALIZARE

- setam capacitatea initiala
- alocam dinamic vectorul de date
- daca esueaza alocarea, capacitatea va deveni 0
- daca nu, returnam coada

```c
queue init_queue(size_t capacity) {

  queue q = {0, 0, capacity, NULL};

  q.data = (queue_data *)malloc(capacity * sizeof(queue_data));

  if (q.data == NULL) {
    if (QUEUE_DEBUG)
      printf("Memory allocation failed\n");

    q.capacity = 0;
    return q;
  }

  q.capacity = capacity;

  if (QUEUE_DEBUG) {
    printf("Initialization successful\n");
    printf("Queue capacity set to: %zu\n", q.capacity);
  }

  return q;
}
```

## EMPTY QUEUE CHECK

- daca capul si finalul cozii sunt egale, coada e goala

```c
int queue_is_empty(queue *q) {
  if (QUEUE_DEBUG && q->head == q->tail)
    printf("queue empty\n");
  return q->head == q->tail;
}
```

## FULL QUEUE CHECK

- daca varful stivei este egal (sau mai mare) decat capacitatea, atunci stiva este plina

```c
int queue_is_full(queue *q) {
  if (QUEUE_DEBUG && q->tail >= q->capacity)
    printf("queue full\n");
  return q->tail >= q->capacity;
}
```

## QUEUE REALLOC

- daca realocarea dinamica este selectata, capacitatea cozii poate fi crescuta cu o valoare data

```c
int queue_realloc(queue *q) {
  queue_data *temp = (queue_data *)realloc(
      q->data, (q->capacity + QUEUE_CHUNK) * sizeof(queue_data));

  if (temp == NULL) {
    if (QUEUE_DEBUG) {
      printf("memory reallocation failed\n");
      printf("queue capacity remains at: %zu\n", q->capacity);
      printf("no enqueue happened\n");
    }
    return 0;
  }

  q->data = temp;
  q->capacity += QUEUE_CHUNK;

  if (QUEUE_DEBUG)
    printf("queue capacity increased to: %zu\n", q->capacity);

  return 1;
}
```

## ENQUEUE

- daca coada este plina si modul dinamic nu este selectat, nu se va face nimic
- daca coada este plina si modul dinamic este selectat, se va face realocare iar in caz de eroare de memorie va returna -1
- daca coada nu este plina, noul element va fi pus la finalul cozii

```c
int enqueue(queue *q, queue_data data) {

  if (queue_is_full(q)) {
    if (QUEUE_DYNAMIC) {
      if (!queue_realloc(q))
        return 0;
    } else {
      if (QUEUE_DEBUG)
        printf("no enqueue happened\n");
      return 0;
    }
  }

  if (QUEUE_DEBUG)
    printf("enqueue: %d\n", data);

  q->data[q->tail++] = data;

  return 1;
}
```

## DEQUEUE

- daca coada este goala, nu se va efectua nicio operatie
- daca coada nu este goala, varful cozii va creste cu 1 (mutam capul cozii)

```c
int dequeue(queue *q) {
  if (queue_is_empty(q)) {
    return 0;
  }

  if (QUEUE_DEBUG) {
    printf("dequeue %d\n", q->data[q->head]);
  }

  q->head++;

  if (q->head == QUEUE_CHUNK) {
    if (QUEUE_DEBUG) {
      printf("chunk reached, resetting head position to 0");
    }

    move_queue(q);
  }

  return 1;
}
```

## QUEUE RESET

- daca din coada se elimina un anumit numar de intrari, aceasta este resetata pe prima pozitie, precum o masina de scris

```c
void move_queue(queue *q) {
  for (size_t i = q->head; i < q->tail; i++) {
    q->data[i - q->head] = q->data[i];
  }
}
```
## HEAD

- daca coada este goala, returneza 0
- daca coada nu este goala, returneaza elementul din capul cozii

```c
queue_data head(queue *q) {

  if (queue_is_empty(q)) {
    return 0;
  }

  if (QUEUE_DEBUG)
    printf("head: %d\n", q->data[q->head]);

  return q->data[q->head];
}
```

## TAIL

- daca coada este goala, returneza 0
- daca coada nu este goala, returneaza elementul din finalul cozii

```c
queue_data tail(queue *q) {

  if (queue_is_empty(q)) {
    return 0;
  }

  if (QUEUE_DEBUG)
    printf("tail: %d\n", q->data[q->tail - 1]);

  return q->data[q->tail - 1];
}
```

## PRINT

```c
void print_queue(queue *q) {
  for (size_t i = q->head; i < q->tail; i++) {
    printf("%d ", q->data[i]);
  }
}
```

## FREE

- elibereaza datele din coada si o initializeaza la loc

```c
void free_queue(queue *q) {
  if (QUEUE_DEBUG)
    printf("free queue\n");

  free(q->data);
  *q = (queue){0, 0, 0, NULL};
}
```