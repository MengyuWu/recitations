CC = gcc
CFLAGS = -Wall -g
LDFLAGS = -g

default: main leaky

main: mymath.o

main.o: mymath.h

mymath.o: mymath.h


.PHONY: clean all valgrind
clean:
	rm -rf *.o main a.out main leaky *.dSYM

all: clean default 

valgrind: main
	valgrind --leak-check=yes ./main
