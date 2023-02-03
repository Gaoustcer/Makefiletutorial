.PHONY:all
vpath %.h include
vpath %.cpp src
vpath %.o build
all: main
main: main.o head.o 
	g++ main.o head.o -o main
# g++ -MM main.cpp -Iinclude
include main.dd
	@echo $?