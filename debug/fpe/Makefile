.PHONY: default help preprocessing assembler object executable all clean
CC = gcc

SOURCE = $(wildcard *.c)
PREPROCESSED= $(patsubst %.c, %.pp_c, $(SOURCE))
ASSEMBLER= $(patsubst %.c, %.asm, $(SOURCE))
OBJECTS = $(patsubst %.c, %.o, $(SOURCE))
EXECUTABLES = $(patsubst %.c, %.e, $(SOURCE))

default: help

help: 	
	@echo "\
Options:\n\n\
  make preprocessing: gcc runs up to pre-processor and stops, creating calculator.pp_c\n\
  make assembler:     gcc runs up to assembler code generation and stops, creating calculator.asm\n\
  make object:        gcc runs up to the creation of the binary object calculator.o\n\
  make executable:    gcc goes all the way and creates the executable calculator.e\n\
  make all:           build all previous\n\
  make clean:         delete output files\n\
  make help:          display this help"

preprocessing: $(PREPROCESSED)

assembler: $(ASSEMBLER)

object: $(OBJECTS)

executable: $(EXECUTABLES)

all: preprocessing assembler object executable

%.pp_c: %.c
	$(CC) -E $^ -o $@ 

%.asm: %.c
	$(CC) -masm=intel -S $^ -o $@ 

%.o: %.c
	$(CC) -c $^ -o $@

%.e: $(OBJECTS)
	$(CC) $^ -o $@

clean:
	rm -rfv $(PREPROCESSED) $(ASSEMBLER) $(OBJECTS) $(EXECUTABLES)
