This introduction is adapted from the gdb tutorial, see original at:

https://www.cs.cmu.edu/~gilpin/tutorial/

Running
=======
The github repo has a C++ file that has a crash during execution.

After downloading the files to your machine, run "make" and the executable file main is created. Run main on gdb (llbd on macosx).

$ make
g++ -ggdb -Wall -o main main.cc

Debugging
=========

Now run the executable on gdb.

$ gdb main
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-110.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/corp.roku/dwong/user/main...done.
(gdb) run
Starting program: /home/corp.roku/dwong/user/main 
warning: Unable to find dynamic linker breakpoint function.
GDB will be unable to debug shared library initializers
and track explicitly loaded dynamic code.
warning: Could not load shared library symbols for 5 libraries, e.g. /lib64/libstdc++.so.6.
Use the "info sharedlibrary" command to see the complete listing.
Do you need "set solib-search-path" or "set sysroot"?
Creating Node, 1 are in existence right now
Creating Node, 2 are in existence right now
Creating Node, 3 are in existence right now
Creating Node, 4 are in existence right now
The fully created list is:
4
3
2
1

Now removing elements:
Creating Node, 5 are in existence right now
Destroying Node, 4 are in existence right now
4
3
2
1


Program received signal SIGSEGV, Segmentation fault.
0x0000000000400ee4 in Node<int>::next (this=0x0) at main.cc:30
30	  Node<T>* next () const { return next_; }
(gdb) 

Inspecting crashes
==================

We saw there was a crash on line 30 of main.cc, this=0x0. In order to debug, it is nice to know who called this function and examine the values in the calling methods. At the gdb prompt, we can type backtrace (bt for short).

(gdb) backtrace
#0  0x0000000000400ee4 in Node<int>::next (this=0x0) at main.cc:30
#1  0x0000000000400dfc in LinkedList<int>::remove (this=0x603010, item_to_remove=@0x7fffffffe18c: 1) at main.cc:79
#2  0x0000000000400abe in main (argc=1, argv=0x7fffffffe298) at main.cc:122

We can see more information, including the parameter that was passed in. The item_to remove parameter in LinkedList<int>::remove() is shown in the backtrace. Traditionally, this location can be examined using x command

(gdb) x 0x7fffffffe18c
0x7fffffffe18c:	0x00000001
(gdb) 

Conditional breakpoint
======================

Now that we know where the program crashed (segfault), we want to see what happened right before the program crashed. A breakpoint can be set to stop the program at a certain point in the program.

(gdb) break LinkedList<int>::remove
Breakpoint 1 at 0x400ca7: file main.cc, line 54.
(gdb) 

To add a condition, it will break only when a condition is satisfied, we are interested in item_to_remove==1

(gdb) condition 1 item_to_remove==1
(gdb) 

This says "Only break at Breapoint 1 if the value of item_to_remove is 1"

Run until breakpoint and single-step
====================================

Now run the program, it will stop at breakpoint 1. After that, single-step until the program reaches the line where the crash happens.

(gdb) run
Starting program: /home/corp.roku/dwong/user/main 
warning: Unable to find dynamic linker breakpoint function.
GDB will be unable to debug shared library initializers
and track explicitly loaded dynamic code.
warning: Could not load shared library symbols for 5 libraries, e.g. /lib64/libstdc++.so.6.
Use the "info sharedlibrary" command to see the complete listing.
Do you need "set solib-search-path" or "set sysroot"?
Creating Node, 1 are in existence right now
Creating Node, 2 are in existence right now
Creating Node, 3 are in existence right now
Creating Node, 4 are in existence right now
The fully created list is:
4
3
2
1

Now removing elements:
Creating Node, 5 are in existence right now
Destroying Node, 4 are in existence right now
4
3
2
1


Breakpoint 1, LinkedList<int>::remove (this=0x603010, item_to_remove=@0x7fffffffe18c: 1) at main.cc:54
54	    Node<T> *marker = head_;
(gdb) step
55	    Node<T> *temp = 0;  // temp points to one behind as we iterate
(gdb) step
57	    while (marker != 0) {
(gdb) step
58	      if (marker->value() == item_to_remove) {
(gdb) step
Node<int>::value (this=0x6030b0) at main.cc:32
32	  const T& value () const { return value_; }
(gdb) step
LinkedList<int>::remove (this=0x603010, item_to_remove=@0x7fffffffe18c: 1) at main.cc:77
77	      marker = 0;  // reset the marker
(gdb) step
78	      temp = marker;
(gdb) step
79	      marker = marker->next();
(gdb) step
Node<int>::next (this=0x0) at main.cc:30
30	  Node<T>* next () const { return next_; }
(gdb)  step

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400ee4 in Node<int>::next (this=0x0) at main.cc:30
30	  Node<T>* next () const { return next_; }
(gdb) 

The step command doesn't need to be entered every time, the enter key will execute the previous command. If you don't want to enter the function, use command next.

The debugging session shows the error is in line 75, the bug can be avoided by simply removing this line.

There is a memory leak in this program, see LinkedList<T>::remove() function. In the original text, it is left as an exercise to the reader.

gdb can be exited by typing quit.
