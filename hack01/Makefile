src = authorize.c
slib = libauthpass.so
target = docrypt

CC = gcc
CPP_FLAGS = -Wall
C_FLAGS = --shared -m32

$(slib) : $(src)
	$(CC) $(C_FLAGS) $(CPP_FLAGS) $^ -o $@

clean:
	rm $(slib)

run:
	LD_PRELOAD=$(slib) ./$(target) $(FILE) $(KEY) 
