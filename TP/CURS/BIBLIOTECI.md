Contine
- directive de typedef / tipuri de data
- declarari de functii


Fisierul .c -> client pentru o biblioteca .h (header)
Biblioteca -> inclusa in fisierul .c principal


Incluziunea de 2 ori a headerului -> redeclararea tipurilor de date / functiilor
- se folosesc Guard-uri

```c
#ifndef __MYHEADER_H

#define __MYHEADER_H

typedef unsigned Element_t;

typedef enum {STIVA_OK, STIVA_FULL, STIVA_EMPTY} Status_Code_t;

typedef struct Stack_t * Stiva_t; // acces "public" la o structura de cod "privata"

Status_Code_t makeStack(Stiva_t *st);
Status_Code_t push(Stiva_t, Element_t);
Status_Code_t pop(Stiva_t, Element_t *);

#endif
```

```c
// MAIN

#include "stiva.h"

int main(void)
{
	Stiva_t st;
}
```