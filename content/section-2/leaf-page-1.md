---
weight: 1
---

# Leaf Page 1

## Verbaque rapite temptabat vacantem

Quantumque per urbem senectam, nisi forti novissima adicit fluvialis aera.
Semina mirator: osculaque Capreas verbis credas fecundaque post id, ad cum ego
sensit. Nece quod pondere terras infamis necari essem **longa dicite** augur et
ramos: decutit ut. [Imagine](#si-hac-venit-vota) per natorum nudis libravit
fraterna, florentia dubie reminiscor septem, corpore? Terram nymphas sentit
alii.

```go
package main

import "fmt"

// this is a comment

func main() {
	fmt.Println("hello world!")
}
```

Some C:

```c
#include <u.h>
#include <libc.h>

void
main(int argc, char *argv[])
{
	int nflag;
	int i, len;
	char *buf, *p;

	nflag = 0;
	if(argc > 1 && strcmp(argv[1], "-n") == 0)
		nflag = 1;

	len = 1;
	for(i = 1+nflag; i < argc; i++)
		len += strlen(argv[i])+1;

	buf = malloc(len);
	if(buf == 0)
		exits("no memory");

	p = buf;
	for(i = 1+nflag; i < argc; i++){
		strcpy(p, argv[i]);
		p += strlen(p);
		if(i < argc-1)
			*p++ = ' ';
	}

	if(!nflag)
		*p++ = '\n';

	if(write(1, buf, p-buf) < 0)
		fprint(2, "echo: write error: %r\n");

	exits((char *)0);
}
```

Some Python:

```py
import datetime as dt

class Timer:
    def __init__(self):
        self._start = 0
        self._elapsed = 0

    @abc.abstractmethod
    def firefox(self):
        raise NotImplementedError

    def duration(self):
        mins = self._elapsed//60
        secs = self._elapsed%60
        return f"{mins}m {secs}s"

text = input('Type a number, and its factorial will be printed: ')
n = int(text)

if n < 0:
    raise ValueError('You must enter a non-negative integer')

factorial = 1
for i in range(2, n + 1):
    factorial *= i

print(factorial)
```

And some YAML:

```yml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - hello.yml
  - hello-svc.yml
```
