main: main.o test.o
	echo "compile final target"
	g++ main.o test.o -o main
main.o: main.cpp
	echo "compile main"
	g++ main.cpp -c -o main.o
test.o: test.h test.cpp
	echo "compile test"
	g++ test.cpp -c -o test.o