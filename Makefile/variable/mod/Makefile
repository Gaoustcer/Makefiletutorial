.PHONY: all random
all:head.o
random:
	@echo $$$$;
	@echo $$$$
%.d: %.cpp
	@set -e;rm -f $@;
	$(CC) -MM $< > $@.$$$$;\
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
	rm -f $@.$$$$
%.o:%.cpp %.h
	@echo $*.cpp
	g++ $*.cpp -c -o $@