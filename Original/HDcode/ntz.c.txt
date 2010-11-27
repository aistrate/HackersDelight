// This has the programs for computing the number of trailing zeros
// in a word.
// Max line length is 57, to fit in hacker.book.
#include <stdio.h>
#include <stdlib.h>     //To define "exit", req'd by XLC.

int nlz(unsigned x) {
   int pop(unsigned x);

   x = x | (x >> 1);
   x = x | (x >> 2);
   x = x | (x >> 4);
   x = x | (x >> 8);
   x = x | (x >>16);
   return pop(~x);
}

int pop(unsigned x) {
   x = x - ((x >> 1) & 0x55555555);
   x = (x & 0x33333333) + ((x >> 2) & 0x33333333);
   x = (x + (x >> 4)) & 0x0F0F0F0F;
   x = x + (x << 8);
   x = x + (x << 16);
   return x >> 24;
}

int ntz1(unsigned x) {
   return 32 - nlz(~x & (x-1));
}

int ntz2(unsigned x) {
   return pop(~x & (x - 1));
}

int ntz3(unsigned x) {
   int n;

   if (x == 0) return(32);
   n = 1;
   if ((x & 0x0000FFFF) == 0) {n = n +16; x = x >>16;}
   if ((x & 0x000000FF) == 0) {n = n + 8; x = x >> 8;}
   if ((x & 0x0000000F) == 0) {n = n + 4; x = x >> 4;}
   if ((x & 0x00000003) == 0) {n = n + 2; x = x >> 2;}
   return n - (x & 1);
}

int ntz4(unsigned x) {
   unsigned y;
   int n;

   if (x == 0) return 32;
   n = 31;
   y = x <<16;  if (y != 0) {n = n -16;  x = y;}
   y = x << 8;  if (y != 0) {n = n - 8;  x = y;}
   y = x << 4;  if (y != 0) {n = n - 4;  x = y;}
   y = x << 2;  if (y != 0) {n = n - 2;  x = y;}
   y = x << 1;  if (y != 0) {n = n - 1;}
   return n;
}

int ntz4a(unsigned x) {
   unsigned y;
   int n;

   if (x == 0) return 32;
   n = 31;
   y = x <<16;  if (y != 0) {n = n -16;  x = y;}
   y = x << 8;  if (y != 0) {n = n - 8;  x = y;}
   y = x << 4;  if (y != 0) {n = n - 4;  x = y;}
   y = x << 2;  if (y != 0) {n = n - 2;  x = y;}
   n = n - ((x << 1) >> 31);
   return n;
}

int ntz5(char x) {
   if (x & 15) {
      if (x & 3) {
         if (x & 1) return 0;
         else return 1;
      }
      else if (x & 4) return 2;
      else return 3;
   }
   else if (x & 0x30) {
      if (x & 0x10) return 4;
      else return 5;
   }
   else if (x & 0x40) return 6;
   else if (x) return 7;
   else return 8;
}

int ntz6(unsigned x) {
   int n;

   x = ~x & (x - 1);
   n = 0;                       // n = 32;
   while(x != 0) {              // while (x != 0) {
      n = n + 1;                //    n = n - 1;
      x = x >> 1;               //    x = x + x;
   }                            // }
   return n;                    // return n;
}

int ntz6a(unsigned x) {
   int n;

                                   n = 32;
                                   while (x != 0) {
                                      n = n - 1;
                                      x = x + x;
                                   }
                                   return n;
}

/* The next three algorithms are not in HD, but they may be in a future
edition.
   Dean Gaudet's algorithm. To be most useful there must be a good way
to evaluate the C "conditional expression" (a?b:c construction) without
branching. The result of a?b:c is b if a is true (nonzero), and c if a
is false (0).
   For example, a compare to zero op that sets a target GPR to 1 if the
operand is 0, and to 0 if the operand is nonzero, will do it. With this
instruction, the algorithm is entirely branch-free. But the most
interesting thing about it is the high degree of parallelism. All six
lines with conditional expressions can be executed in parallel (on a
machine with sufficient computational units).
   Although the instruction count is 30 measured statically, it could
execute in only 10 cycles on a machine with sufficient parallelism.
   The first two uses of y can instead be x, which would increase the
useful parallelism on most machines (the assignments to y, bz, and b4
could then all run in parallel). */

int ntz7(unsigned x) {
   unsigned y, bz, b4, b3, b2, b1, b0;

   y = x & -x;               // Isolate rightmost 1-bit.
   bz = y ? 0 : 1;           // 1 if y = 0.
   b4 = (y & 0x0000FFFF) ? 0 : 16;
   b3 = (y & 0x00FF00FF) ? 0 : 8;
   b2 = (y & 0x0F0F0F0F) ? 0 : 4;
   b1 = (y & 0x33333333) ? 0 : 2;
   b0 = (y & 0x55555555) ? 0 : 1;
   return bz + b4 + b3 + b2 + b1 + b0;
}

/* Below is David Seal's algorithm, found at
http://www.ciphersbyritter.com/NEWS4/BITCT.HTM Table
entries marked "u" are unused. 6 ops including a
multiply, plus an indexed load. */

#define u 99
int ntz8(unsigned x) {

   static char table[64] =
     {32, 0, 1,12, 2, 6, u,13,   3, u, 7, u, u, u, u,14,
      10, 4, u, u, 8, u, u,25,   u, u, u, u, u,21,27,15,
      31,11, 5, u, u, u, u, u,   9, u, u,24, u, u,20,26,
      30, u, u, u, u,23, u,19,  29, u,22,18,28,17,16, u};

   x = (x & -x)*0x0450FBAF;
   return table[x >> 26];
}

/* Seal's algorithm with multiply expanded.
9 elementary ops plus an indexed load. */

int ntz8a(unsigned x) {

   static char table[64] =     {32, 0, 1,12,  2, 6, u,13,
      3, u, 7, u,  u, u, u,14,  10, 4, u, u,  8, u, u,25,
      u, u, u, u,  u,21,27,15,  31,11, 5, u,  u, u, u, u,
      9, u, u,24,  u, u,20,26,  30, u, u, u,  u,23, u,19,
     29, u,22,18, 28,17,16, u};

   x = (x & -x);
   x = (x << 4) + x;    // x = x*17.
   x = (x << 6) + x;    // x = x*65.
   x = (x << 16) - x;   // x = x*65535.
   return table[x >> 26];
}

/* Reiser's algorithm. Three ops including a "remainder,"
plus an indexed load. */

int ntz9(unsigned x) {

   static char table[37] = {32,  0,  1, 26,  2, 23, 27,
                 u,  3, 16, 24, 30, 28, 11,  u, 13,  4,
                 7, 17,  u, 25, 22, 31, 15, 29, 10, 12,
                 6,  u, 21, 14,  9,  5, 20,  8, 19, 18};

   x = (x & -x)%37;
   return table[x];
}

int errors;
void error(int x, int y) {
   errors = errors + 1;
   printf("Error for x = %08x, got %d\n", x, y);
}

int main() {
   int i, m, n;
   static unsigned test[] = {0,32, 1,0, 2,1, 3,0, 4,2, 5,0,
      6,1, 7,0, 8,3, 9,0, 128,7, 255,0, 256,8, 0xFFFFFFF0,4,
      0x10000000,28, 0x3000FF00,8, 0x40000000,30, 0xC0000000,30,
      0x80000000,31, 0x60000000,29, 0x00011000, 12};

   n = sizeof(test)/4;

   printf("ntz1:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz1(test[i]) != test[i+1]) error(test[i], ntz1(test[i]));}

   printf("ntz2:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz2(test[i]) != test[i+1]) error(test[i], ntz2(test[i]));}

   printf("ntz3:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz3(test[i]) != test[i+1]) error(test[i], ntz3(test[i]));}

   printf("ntz4:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz4(test[i]) != test[i+1]) error(test[i], ntz4(test[i]));}

   printf("ntz4a:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz4a(test[i]) != test[i+1]) error(test[i], ntz4a(test[i]));}

   printf("ntz5:\n");
   for (i = 0; i < n; i += 2) {
      m = test[i+1]; if (m > 8) m = 8;
      if (ntz5(test[i]) != m) error(test[i], ntz5(test[i]));}

   printf("ntz6:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz6(test[i]) != test[i+1]) error(test[i], ntz6(test[i]));}

   printf("ntz6a:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz6a(test[i]) != test[i+1]) error(test[i], ntz6a(test[i]));}

   printf("ntz7:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz7(test[i]) != test[i+1]) error(test[i], ntz7(test[i]));}

   printf("ntz8:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz8(test[i]) != test[i+1]) error(test[i], ntz8(test[i]));}

   printf("ntz8a:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz8a(test[i]) != test[i+1]) error(test[i], ntz8a(test[i]));}

   printf("ntz9:\n");
   for (i = 0; i < n; i += 2) {
      if (ntz9(test[i]) != test[i+1]) error(test[i], ntz9(test[i]));}

   if (errors == 0)
      printf("Passed all %d cases.\n", sizeof(test)/8);
}
