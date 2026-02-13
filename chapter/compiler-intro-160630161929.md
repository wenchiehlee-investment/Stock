# compiler-intro-160630161929

來源：`chapter/compiler-intro-160630161929.pdf`
頁數：99

## 第 1 頁

INTRO TO COMPILER
DEVELOPMENT
LOGAN CHIEN
http://slide.logan.tw/compiler-intro/

## 第 2 頁

LOGAN CHIEN
Soware engineer at MediaTek
Ametuar LLVM/Clang developer
Integrated LLVM/Clang into Android NDK

## 第 3 頁

WHY I LOVE COMPILERS?
I have been a faithful reader of Jserv's blog for ten years.
I was inspired by the compiler and the virtual machine
technologies mentioned in his blog.
I have decided to choose compiler technologies as my
research topic since then.

## 第 4 頁

UNDERGRADUATE COMPILER
COURSE
I took the undergraduate compiler course when I was a
sophomore.

## 第 5 頁

MY PROFESSOR SAID ...
“We took many lectures to discuss about the parser.
However, when people say they are doing compiler
research, with large possibility, they are not referring to the
parsing technique.”

## 第 6 頁

I AM HERE TO ...
Re-introduce the compiler technologies,
Give a lightening talk on industrial-strength compiler
design,
Explain the connection between compiler technologies and
the industry.

## 第 7 頁

AGENDA
Re-introduction to Compiler (30min)
Industrial-strength Compiler Design (90min)
Compiler and ICT Industry (20min)

## 第 8 頁

RE-INTRODUCTION TO
COMPILER

## 第 9 頁

WHAT IS A COMPILER?
Compiler are tools for programmers to translate
programmer's thought into computer runnable programs.
ANALOGY — Translators who turn from one language to
another, such as those who translate Chinese to English.

## 第 10 頁

WHAT HAVE WE LEARNED IN
UNDERGRADUATE COMPILER
COURSE?

## 第 11 頁

LEXER
Reads the input source code (as a sequence of bytes) and
converts them into a stream of tokens.
unsigned background(unsigned foreground) { 
    if ((foreground >> 16) > 0x80) { 
        return 0; 
    } else { 
        return 0xffffff; 
    } 
}
unsigned background ( unsigned foreground ) { if ( ( foreground >>
16 ) > 128 ) { return 0 ; } else { return 16777215 ; } }

## 第 12 頁

PARSER
Reads the tokens and build an AST according to the syntax.
unsigned background ( unsigned foreground ) { if ( ( foreground >>
16 ) > 128 ) { return 0 ; } else { return 16777215 ; } }
(procedure background 
    (args '(foreground)) 
    (compound-stmt 
        (if-stmt 
            (bin-expr GE (bin-expr RSHIFT foreground 16) 128) 
            (return-stmt 0) 
            (return-stmt 16777215))))

## 第 13 頁

CODE GENERATOR
Generate the machine code or (assembly) according to the
AST. In the undergraduate course, we usually simply do
syntax-directed translation.
(procedure background 
    (args '(foreground)) 
    (compound-stmt 
        (if-stmt 
            (bin-expr GE (bin-expr RSHIFT foreground 16) 128) 
            (return-stmt 0) 
            (return-stmt 16777215))))
        lsr w8, w0, #16 
        cmp w8, #128 
        b.lo .Lelse 
        mov w0, wzr 
        ret 
.Lelse: 
        orr w0, wzr, #0xffffff 
        ret

## 第 14 頁

WHAT'S MISSING?
Can a person who can only lex and parse sentences translate
articles well?

## 第 15 頁

COMPILER REQUIREMENTS
A compiler should translate the source code precisely.
A compiler should utilize the device eﬀiciently.

## 第 16 頁

THREE RELATED FIELDS
Programming Language
Computer Architecture
Compiler

## 第 17 頁

PROGRAMMING LANGUAGE THEORY
Essential component of a programming language: type
theory, variable scoping, language semantics, etc.
How do people reason and compose a program?
Create an abstraction that is understandable to human and
tracable to computers.

## 第 18 頁

EXAMPLE: SUBTYPE AND MUTABLE
RECORDS
Why you can't perform following conversion in C++?
void test(int *ptr) { 
  int **p = &ptr; 
  const int** a = p;  // Compiler gives warning 
  // ... 
}
This is related to covariant type and contravarience type. With PLT, we know that we can only choose
two of (a) covariant type, (b) mutable records, and (c) type consistency.
void test(int *ptr) { 
  const int c = 0; 
  int **p = &ptr; 
  const int** a = p;  // If it is allowed, bad programs will pass. 
  *a = &c; 
  *p = 5;  // No warning here.
}

## 第 19 頁

EXAMPLE: DYNAMIC SCOPING IN
BASH
#!/bin/sh 
v=1                  # Initialize v with 1 
foo () { 
  echo "foo:v=${v}"  # Which v is referred? 
  v=2                # Which v is assigned? 
} 
bar () { 
  local v=3 
  foo 
  echo "bar:v=${v}"  # What will be printed? 
} 
v=4                  # Assign 4 to v 
bar 
echo "v=${v}"        # What will be printed?
Ans: foo:v=3, bar:v=2, v=4. Surprisingly, foo is
accessing local v in bar instead of the global v.

## 第 20 頁

EXAMPLE: NON-LEXICAL SCOPING IN
JAVASCRIPT
// Javascript, the bad part 
function bad(v) { 
    var sum; 
    with (v) { 
        sum = a + b; 
    } 
    return sum;
} 
console.log(bad({a: 5, b: 10})); 
console.log(bad({a: 5, b: 10, sum: 100}));
Ans: The second console.log() prints undefined.

## 第 21 頁

COMPUTER ARCHITECTURE
Instruction set architecture: CISC vs. RISC.
Out-of-Order Execution vs. Instruction Scheduling.
Memory hierarchy
Memory model

## 第 22 頁

QUIZ: DO YOU REALLY KNOW C?
Is it guaranteed that v will always be loaded aer pred?
int pred; 
int v; 
int get(int a, int b) { 
    int res; 
    if (pred > 0) { 
        res = v * a - v / b; 
    } else { 
        res = v * a + v / b; 
    } 
    return res;
}
Ans: No. Independent reads/writes can be reordered. The standard only requires the result should
be the same as running from top to bottom (in a single thread.)

## 第 23 頁

COMPILER ANALYSIS
Data-flow analysis — Analyze value ranges, check the
conditions or contraints, figure out modifications to
variables, etc.
Control-flow analysis — Analyze the structure of the
program, such as control dependency and loop structure.
Memory dependency analysis — Analyze the memory
access pattern of the access to array elements or pointer
dereferences, e.g. alias analysis.

## 第 24 頁

ALIAS ANALYSIS
Determine whether two pointers can refers to the same
object or not.
void move(char *dst, const char *src, int n) { 
    for (int i = 0; i < n; ++i) { 
        dst[i] = src[i]; 
    } 
}
int sum(int *ptr, const int *val, int n) { 
    int res = 0; 
    for (int i = 0; i < n; ++i) { 
        res += *val; 
        *ptr++ = 10; 
    } 
    return res;
}

## 第 25 頁

QUIZ: DO YOU KNOW C++?
class QMutexLocker { 
public: 
  union { 
    QMutex *mtx_; 
    uintptr_t val_; 
  }; 
  void unlock() { 
    if (val_) { 
      if ((val_ & (uintptr_t)1) == (uintptr_t)1) { 
        val_ &= ~(uintptr_t)1; 
        mtx_->unlock(); 
      } 
    } 
  } 
};
Pitfall: Reading from union fields that were not written
previously results in undefined behavior. Type-Based Alias
Analysis (TBAA) exploits this rule.

## 第 26 頁

COMPILER OPTIMIZATION
Scalar optimization — Fold the constants, remove the
redundancies, change expressions with identities, etc.
Vector optimization — Convert several scalar operations
into one vector operation, e.g. combining for add instruction
into one vector add.
Interprocedural optimization — function inlining,
devirtualization, cross-function analysis, etc.

## 第 27 頁

OTHER COMPILER-RELATED
TECHNOLOGY
Just-in-time compilers
Binary translators
Program profiling and performance measurement
Facilities to run compiled executables, e.g. garbage
collectors

## 第 28 頁

INDUSTRIAL-
STRENGTH COMPILER
DESIGN

## 第 29 頁

What's the diﬀerence between your
final project and the industrial-
strength compiler?

## 第 30 頁

KEY DIFFERENCE
Analysis — Reasons program structures and changes of
values.
Optimization — Applies several provably correct
transformation which should make program run faster.
Intermediate Representation — Data structure on which
analyses and optimizations are based.

## 第 31 頁

INTERMEDIATE REPRESENTATION1/4
A data structure for program analyses and optimizations.
High-level enough to capture important properties and
encapsulates hardware limitation.
Low-level enough to be analyzed by analyses and
manipulated by transformations.
An abstraction layer for multiple front-ends and back-ends.

## 第 32 頁

INTERMEDIATE REPRESENTATION2/4
C/C++
Ada
Fortran
Go
arm
aarch64
mips
nds
x86
GENERIC RTLGIMPLE
Tree SSA
GCC Compiler Pipeline

## 第 33 頁

INTERMEDIATE REPRESENTATION3/4
C/C++
Swift
Rust
Obj‑C
arm
aarch64
mips
sparc
x86
MachInstrSelDAGLLVM IR
LLVM Compiler Pipeline

## 第 34 頁

INTERMEDIATE REPRESENTATION4/4
GCC — GENERIC, GIMPLE, Tree SSA, and RTL.
LLVM — LLVM IR, Selection DAG, and Machine Instructions.
Java HotSpot — HIR, LIR, and MIR.

## 第 35 頁

CONTROL FLOW GRAPH1/3
Basic block — A sequence of instructions that will only be
entered from the top and exited from the end.
Edge — If the basic block s may branch to t, then we add a
directed edge (s, t).
Predecessor/Successor — If there is an edge (s, t), then s is a
predecessor of t and t is a successor of s.

## 第 36 頁

CONTROL FLOW GRAPH2/3
// y = a * x + b; 
void matmul(double *restrict y, 
            unsigned long m, unsigned long n, 
            const double *restrict a, 
            const double *restrict x, 
            const double *restrict b) { 
  for (unsigned long r = 0; r < m; ++r) { 
    double t = b[r]; 
    for (unsigned long i = 0; i < n; ++i) { 
      t += a[r * n + i] * x[i]; 
    } 
    y[r] = t; 
  } 
}
Input source program

## 第 37 頁

CONTROL FLOW GRAPH3/3
b p   =  g e t e l e m e n t p t r  b ,  r
t    =  l o a d  b p
i    =  0
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 1
B 2
r n   =  m u l  r ,  n
a i   =  a d d  r n ,  i
a p   =  g e t e l e m e n t p t r  a ,  a i
a d   =  l o a d  a p
x p   =  g e t e l e m e n t p t r  x ,  i
x d   =  l o a d  x p
a x   =  m u l  a d ,  x d
t    =  a d d  t ,  a x
i    =  a d d  i ,  1
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 3
y p   =  g e t e l e m e n t p t r  y ,  r
s t o r e  t ,  y p
r    =  a d d  r ,  1
c m p  =  i c m p  l t ,  r ,  n
b r  c m p ,  B 1 ,  B 4B 4
r e t  v o i d
B 0  ( E N T R Y )
r    =  0
c m p  =  i c m p  l t  r ,  m
b r  c m p ,  B 1 ,  B 4

## 第 38 頁

VARIABLE DEFINITION
b p   =  g e t e l e m e n t p t r  b ,  r
t    =  l o a d  b p
i    =  0
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 1
B 2
r n   =  m u l  r ,  n
a i   =  a d d  r n ,  i
a p   =  g e t e l e m e n t p t r  a ,  a i
a d   =  l o a d  a p
x p   =  g e t e l e m e n t p t r  x ,  i
x d   =  l o a d  x p
a x   =  m u l  a d ,  x d
t    =  a d d  t ,  a x
i    =  a d d  i ,  1
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 3
y p   =  g e t e l e m e n t p t r  y ,  r
s t o r e  t ,  y p
r    =  a d d  r ,  1
c m p  =  i c m p  l t ,  r ,  n
b r  c m p ,  B 1 ,  B 4B 4
r e t  v o i d
B 0  ( E N T R Y )
r    =  0
c m p  =  i c m p  l t  r ,  m
b r  c m p ,  B 1 ,  B 4
definitions of r
The place where a variable is assigned or defined.

## 第 39 頁

VARIABLE USE
b p   =  g e t e l e m e n t p t r  b ,  r
t    =  l o a d  b p
i    =  0
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 1
B 2
r n   =  m u l  r ,  n
a i   =  a d d  r n ,  i
a p   =  g e t e l e m e n t p t r  a ,  a i
a d   =  l o a d  a p
x p   =  g e t e l e m e n t p t r  x ,  i
x d   =  l o a d  x p
a x   =  m u l  a d ,  x d
t    =  a d d  t ,  a x
i    =  a d d  i ,  1
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 3
y p   =  g e t e l e m e n t p t r  y ,  r
s t o r e  t ,  y p
r    =  a d d  r ,  1
c m p  =  i c m p  l t ,  r ,  n
b r  c m p ,  B 1 ,  B 4B 4
r e t  v o i d
B 0  ( E N T R Y )
r    =  0
c m p  =  i c m p  l t  r ,  m
b r  c m p ,  B 1 ,  B 4
definitions of r
uses of r
The places where a variable is referred or used.

## 第 40 頁

REACHING DEFINITION1/2
b p   =  g e t e l e m e n t p t r  b ,  r
t    =  l o a d  b p
i    =  0
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 1
B 2
r n   =  m u l  r ,  n
a i   =  a d d  r n ,  i
a p   =  g e t e l e m e n t p t r  a ,  a i
a d   =  l o a d  a p
x p   =  g e t e l e m e n t p t r  x ,  i
x d   =  l o a d  x p
a x   =  m u l  a d ,  x d
t    =  a d d  t ,  a x
i    =  a d d  i ,  1
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 3
y p   =  g e t e l e m e n t p t r  y ,  r
s t o r e  t ,  y p
r    =  a d d  r ,  1
c m p  =  i c m p  l t ,  r ,  n
b r  c m p ,  B 1 ,  B 4B 4
r e t  v o i d
B 0  ( E N T R Y )
r    =  0
c m p  =  i c m p  l t  r ,  m
b r  c m p ,  B 1 ,  B 4
{d0} {d0, d1}
{d0, d1}
d0:
d1:
{d1}
{d0, d1}
{d0, d1}
Definitions that reaches a use.

## 第 41 頁

REACHING DEFINITION2/2
Constant propagation is a good example to show the
usefulness of reaching definition.
void test(int cond) { 
    int a = 1;  // d0 
    int b = 2;  // d1 
    if (cond) { 
        c = 3;  // d2 
    } else { 
        // ReachDef[a] = {d0} 
        // ReachDef[b] = {d1} 
        c = a + b;  // d3 
    } 
    // ReachDef[c] = {d2, d3} 
    use(c); 
}

## 第 42 頁

DOMINANCE RELATION1/2
A basic block s dominates t iﬀ every paths that goes from
entry to t will pass through s.
Every basic block in a CFG has an immediate dominator and
forms a dominator tree.

## 第 43 頁

DOMINANCE RELATION2/2
B0
B1
B2
B3
B4
B0
B1
B2B3
B4
Dominator Tree

## 第 44 頁

DOMINANCE FRONTIER
A basic block t is a dominance frontier of a basic block s, if
one of predecessor of t is dominated by s but t is not strictly
dominated by s.
B0
B1 B2
B3
B6
B4
B5
DF[B2] = {B3, B5}
DF[B4] = {B4}
DF[B1] = {B3, B5}

## 第 45 頁

STATIC SINGLE-ASSIGNMENT FORM
static — A static analysis to the program (not the execution.)
single-assignment — Every variable can only be assigned
once.
SSA form is the most popular intermediate representation
recently.
It is adopted by a wide range of compilers, such as GCC,
LLVM, Java HotSpot, Android ART, etc.

## 第 46 頁

SSA PROPERTIES
Every variables can only be defined once.
Every uses can only refer to one definition.
Use phi function to handle the merged control-flow.

## 第 47 頁

PHI FUNCTIONS
define void @foo(i1 cond, 
                 i32 a, i32 b) { 
ent: 
    br cond, b1, b2 
b1: 
    t0 = mul a, 4 
    br b3 
b2: 
    t1 = mul b, 5 
    br b3 
b3: 
    t2 = phi (t0), (t1) 
    use(t2) 
    ret 
}
define void @foo(i32 n) { 
ent: 
    br loop 
loop: 
    i0 = phi (0), (i1) 
    cmp = icmp ge i0, n 
    br cmp, end, cont 
cont: 
    use(i0) 
    i1 = add i0, 1 
    br loop 
end: 
    ret 
}

## 第 48 頁

ADVANTAGE OF SSA FORM
Compact — Reduce the def-use chain.
Referential transparency — The properties associated with a
variable will not be changed, aka. context-free.

## 第 49 頁

REDUCED DEF-USE CHAIN
void foo(int cond1, int cond2, 
         int a, int b) { 
    int t; 
    if (cond1) { 
        t = a * 4;  // d0 
    } else { 
        t = b * 5;  // d1 
    } 
    if (cond2) { 
        // reach-def: {d0, d1} 
        use(t); 
    } else { 
        // reach-def: {d0, d1} 
        use(t); 
    } 
}
void foo(int cond1, int cond2, 
         int a, int b) { 
    if (cond1) { 
        t.0 = a * 4; 
    } else { 
        t.1 = b * 5; 
    } 
    t.2 = phi(t.0, t.1); 
    if (cond2) { 
        use(t.2); 
    } else { 
        use(t.2); 
    } 
}

## 第 50 頁

REFERENTIAL TRANSPARENCY1/2
void foo() { 
    int r = 5;  // d0 
    // ... SOME CODE ... 
    // We can only assume 
    // "r == 5" if d0 is the 
    // only reaching definition. 
    use(r); 
}
void foo() { 
    r.0 = 5; 
    // ... SOME CODE ... 
    // No matter what code are 
    // skipped above, it is safe 
    // to replace following r.0 
    // with 5. 
    use(r.0); 
}

## 第 51 頁

REFERENTIAL TRANSPARENCY2/2
void foo(int a, int b) { 
    int c = a + b; 
    int r = a + b; 
    // Can we simply replace 
    // all occurrence of r with 
    // c?  (NO)
    // ... SOME CODE ... 
    use(r); 
}
void foo(int a, int b) { 
    c.0 = a + b; 
    r.0 = a + b; 
    // ... SOME CODE ... 
    // No matter what code are 
    // skipped above, it is safe 
    // to replace following r.0 
    // with c.0. 
    use(r.0); 
}

## 第 52 頁

BUILDING SSA FORM
1. Compute domination relationships between basic blocks
and build the dominator tree.
2. Compute dominator frontiers.
3. Insert phi functions at dominator frontiers.
4. Traverse the dominator tree and rename the variables.

## 第 53 頁

BEFORE SSA CONSTRUCTION
b p   =  g e t e l e m e n t p t r  b ,  r
t    =  l o a d  b p
i    =  0
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 1
B 2
r n   =  m u l  r ,  n
a i   =  a d d  r n ,  i
a p   =  g e t e l e m e n t p t r  a ,  a i
a d   =  l o a d  a p
x p   =  g e t e l e m e n t p t r  x ,  i
x d   =  l o a d  x p
a x   =  m u l  a d ,  x d
t    =  a d d  t ,  a x
i    =  a d d  i ,  1
c m p  =  i c m p  l t ,  i ,  n
b r  c m p ,  B 2 ,  B 3
B 3
y p   =  g e t e l e m e n t p t r  y ,  r
s t o r e  t ,  y p
r    =  a d d  r ,  1
c m p  =  i c m p  l t ,  r ,  n
b r  c m p ,  B 1 ,  B 4B 4
r e t  v o i d
B 0  ( E N T R Y )
r    =  0
c m p  =  i c m p  l t  r ,  m
b r  c m p ,  B 1 ,  B 4

## 第 54 頁

AFTER SSA CONSTRUCTION
r . 0  =  p h i  ( 0 ,  B 0 )  ( r . 1  B 3 )
b p   =  g e t e l e m e n t p t r  b ,  r . 0
t . 0   =  l o a d  b p
c m p  =  i c m p  l t ,  0 ,  n
b r  c m p ,  B 2 ,  B 3
B 1
B 2
t . 1  =  p h i  ( t . 0 ,  B 1 )  ( t . 2  B 2 )
i . 0  =  p h i  ( 0 ,  B 1 )  ( i . 1 ,  B 2 )
r n   =  m u l  r . 0 ,  n
a i   =  a d d  r n ,  i . 1
a p   =  g e t e l e m e n t p t r  a ,  a i
a d   =  l o a d  a p
x p   =  g e t e l e m e n t p t r  x ,  i . 1
x d   =  l o a d  x p
a x   =  m u l  a d ,  x d
t . 2  =  a d d  t . 1 ,  a x
i . 1  =  a d d  i . 0 ,  1
c m p  =  i c m p  l t ,  i . 1 ,  n
b r  c m p ,  B 2 ,  B 3
B 3
t . 3  =  p h i  ( t 0 ,  B 1 )  ( t . 2 ,  B 2 )
y p   =  g e t e l e m e n t p t r  y ,  r . 0
s t o r e  t . 3 ,  y p
r . 1  =  a d d  r . 0 ,  1
c m p  =  i c m p  l t ,  r . 1 ,  n
b r  c m p ,  B 1 ,  B 4B 4
r e t  v o i d
B 0  ( E N T R Y )
c m p  =  i c m p  l t  0 ,  m
b r  c m p ,  B 1 ,  B 4

## 第 55 頁

OPTIMIZATIONS

## 第 56 頁

CONSTANT PROPAGATION1/3
Constant propagation, sometimes known as constant
folding, will evaluate the instructions with constant
operands and propagate the constant result.
a = add 2, 3 
b = a 
c = mul a, b
a = 5 
b = 5 
c = 25

## 第 57 頁

CONSTANT PROPAGATION2/3
Why do we need constant propagation?
struct queue *create_queue() {
    return (struct queue*)malloc(sizeof(struct queue *) * 16); 
}
int process_data(int a, int b, int c) { 
    int k[] = { 0x13, 0x17, 0x19 }; 
    if (DEBUG) { 
        verify_data(k, data); 
    } 
    return (a * k[1] * k[2] + b * k[0] * k[2] + c * k[0] * k[1]); 
}

## 第 58 頁

CONSTANT PROPAGATION3/3
For each basic block b from CFG in reversed postorder:
For each instruction i in basic block b from top to bottom:
If all of its operands are constants, the operation has no
side-eﬀect, and we know how to evaluate it at compile time,
then evaluate the result, remove the instruction, and replace
all usages with the result.

## 第 59 頁

GLOBAL VALUE NUMBERING1/2
Global value numbering tries to give numbers to the
computed expression and maps the newly visited expression
to the visited ones.
a = c * d;  // [(*, c, d) -> a] 
e = c;      // [(*, c, d) -> a] 
f = e * d;  // query: is (*, c, d) available? 
use(a, e, f);
a = c * d; 
use(a, c, a);

## 第 60 頁

GLOBAL VALUE NUMBERING2/2
Traverse the basic blocks in depth-first order on dominator
tree.
Maintain a stack of hash table. Once we have returned from
a child node on the dominator tree, then we have to pop the
stack top.
Visit the instructions in the basic block with tﾠ=ﾠopﾠa,ﾠb
form and compute the hash for (op, a, b).
If (op, a, b) is already in the hash table, then change 
tﾠ=ﾠopﾠa,ﾠb with tﾠ=ﾠhash_tab[(op,ﾠa,ﾠb)]
Otherwise, insert (op, a, b) -> t  to the hash table.

## 第 61 頁

Otherwise, insert  to the hash table.
DEAD CODE ELIMINATION1/6
Dead code elimination (DCE) removes unreachable
instructions or ignored results.
Constant propagation might reveal more dead code since
the branch conditions become constant value.
On the other hand, DCE can exploit more constant for
constant propagation because several definitions are
removed from the program.

## 第 62 頁

DEAD CODE ELIMINATION2/6
Conditional statements with constant condition
if (kIsDebugBuild) {              // Dead code 
    check_invariant(a, b, c, d);  // Dead code 
}

## 第 63 頁

DEAD CODE ELIMINATION3/6
Platform-specific constant
void hash_ent_set_val(struct hash_ent *h, int v) { 
    if (sizeof(int) <= sizeof(void *)) { 
        h->p = (void *)(uintptr_t)(v); 
    } else { 
        h->p = malloc(sizeof(int));  // Dead code 
        *(h->p) = v;                 // Dead code 
    } 
}

## 第 64 頁

DEAD CODE ELIMINATION4/6
Computed result ignored
int compute_sum(int a, int b) { 
    int sum = (a + b) * (a - b + 1) / 2;  // Dead code: Not used 
    return 0; 
}
Dead code aer dead store optimization
void test(int *p, bool cond, int a, int b, int c) { 
    if (cond) { 
        t = a + b;  // Dead code 
        *p = t;     // Will be removed by DSE 
    } 
    *p = c; 
}

## 第 65 頁

DEAD CODE ELIMINATION5/6
Dead code aer code specialization
void matmul(double *restrict y, unsigned long m, unsigned long n, 
            const double *restrict a, 
            const double *restrict x, 
            const double *restrict b) { 
    if (n == 0) { 
        for (unsigned long r = 0; r < m; ++r) { 
            double t = b[r]; 
            for (unsigned long i = 0; i < n; ++i) {  // Dead code 
                t += a[r * n + i] * x[i];            // Dead code 
            }                                        // Dead code 
            y[r] = t; 
        } 
    } else { 
        // ... skipped ... 
    } 
}

## 第 66 頁

DEAD CODE ELIMINATION6/6
Traverse the CFG starting from entry in reversed post order.
Only traverse the successor that may be visited, i.e. if the
branch condition is a constant, then ignore the other side.
While traversing the basic block, clear the dead instructions
within the basic block.
Aer the traversal, remove the unvisited basic blocks and
remove the variable uses that refers to the variables that are
defined in the unvisited basic blocks.

## 第 67 頁

LOOP OPTIMIZATIONS
It is reasonable to assume a program spends more time in
the loop body. Thus, loop optimization is an important issue
in the compiler.

## 第 68 頁

LOOP INVARIANT CODE MOTION1/3
Loop invariant code motion (LICM) is an optimization which
moves loop invariants or loop constants out of the loop.
int test(int n, int a, int b, int c) { 
    int sum = 0; 
    for (int i = 0; i < n; ++i) { 
        sum += i * a * b * c;  // a*b*c is loop invariant 
    } 
    return sum;
}

## 第 69 頁

LOOP INVARIANT CODE MOTION2/3
for (unsigned long r = 0; r < m; ++r) { 
    double t = b[r]; 
    for (unsigned long i = 0; i < n; ++i) { 
        t += a[(r * n) + i] * x[i];  // "r*n" is a inner loop invariant
    } 
    y[r] = t; 
}
for (unsigned long r = 0; r < m; ++r) { 
    double t = b[r]; 
    unsigned long k = r * n;  // "r*n" moved out of the inner loop 
    for (unsigned long i = 0; i < n; ++i) { 
        t += a[k + i] * x[i]; 
    } 
    y[r] = t; 
}

## 第 70 頁

LOOP INVARIANT CODE MOTION3/3
How do we know whether a variable is a loop invariant?
If the computation of a value is not (transitively) depending
on following black lists, we can assume such value is a loop
invariant:
Phi instructions associating with the loop being
considered
Instructions with side-eﬀects or non-pure instructions,
e.g. function calls

## 第 71 頁

INDUCTION VARIABLE
If x represents the trip count (iteration count), then we call
a*x+b induction variables.
for (unsigned long r = 0; r < m; ++r) { 
    double t = b[r]; 
    unsigned long k = r * n;  // outer loop IV 
    for (unsigned long i = 0; i < n; ++i) { 
        unsigned long m = k + i;  // inner loop IV 
        t += a[m] * x[i]; 
    } 
    y[r] = t; 
}
k and r are induction variables of outer loop.
i and m are induction variables of inner loop.

## 第 72 頁

LOOP STRENGTH REDUCTION1/2
Strength reduction is an optimization to replace the
multiplication in induction variables with an addition.
for (unsigned long r = 0; r < m; ++r) { 
    double t = b[r]; 
    unsigned long k = r * n;  // outer loop IV 
    for (unsigned long i = 0; i < n; ++i) { 
        unsigned long m = k + i;  // inner loop IV 
        t += a[m] * x[i]; 
    } 
    y[r] = t; 
}
for (unsigned long r = 0, k = 0; r < m; ++r, k += n) {  // k
    double t = b[r]; 
    for (unsigned long i = 0, m = k; i < n; ++i, ++m) {  // m 
        t += a[m] * x[i]; 
    } 
    y[r] = t; 
}

## 第 73 頁

LOOP STRENGTH REDUCTION2/2
We can even rewrite the range to eliminate the index
computation.
int sum(const int *a, int n) { 
    int res = 0; 
    for (int i = 0; i < n; ++i) { 
        res += a[i];  // implicit *(a + sizeof(int) * i) 
    } 
    return res;
}
int sum(const int *a, int n) { 
    int res = 0; 
    const int *end = a + n;  // implicit a + sizeof(int) * n 
    for (const int *p = a; p != end; ++p) {  // range updated 
        res += *p; 
    } 
    return res;
}

## 第 74 頁

LOOP UNROLLING1/2
Unroll the loop body multiple times.
Purpose: Reduce amortized loop iteration overhead.
Purpose: Reduce load/store stall and exploit instruction-
level parallelism, e.g. soware pipelining.
Purpose: Prepare for vectorization, e.g. SIMD.

## 第 75 頁

LOOP UNROLLING2/2
for (unsigned long r = 0, k = 0; r < m; ++r, k += n) { 
    double t = b[r]; 
    switch (n & 0x3) {  // Duff's device 
        case 3: t += a[k + 2] * x[2]; 
        case 2: t += a[k + 1] * x[1]; 
        case 1: t += a[k] * x[0]; 
    } 
    for (unsigned long i = n & 0x3; i < n; i += 4) { 
        t += a[k + i] * x[i]; 
        t += a[k + i + 1] * x[i + 1]; 
        t += a[k + i + 2] * x[i + 2]; 
        t += a[k + i + 3] * x[i + 3]; 
    } 
    y[r] = t; 
}

## 第 76 頁

INSTRUCTION SELECTION1/5
Instruction selection is a process to map IR into machine
instructions.
Compiler back-ends will perform pattern matching to the
best select instructions (according to the heuristic.)

## 第 77 頁

INSTRUCTION SELECTION2/5
Complex operations, e.g. shi-and-add or multiply-
accumulate.
Array load/store instructions, which can be translated to one
shi-add-and-load on some architectures.
IR instructions are which not natively supported by the target
machine.

## 第 78 頁

INSTRUCTION SELECTION3/5
define i64 @mac(i64 %a, i64 %b, i64 %c) { 
ent: 
  %0 = mul i64 %a, %b 
  %1 = add i64 %0, %c 
  ret i64 %1 
}
mac: 
 madd x0, x1, x0, x2 
 ret

## 第 79 頁

INSTRUCTION SELECTION4/5
define i64 @load_shift(i64* %a, i64 %i) { 
ent: 
  %0 = getelementptr i64, i64* %a, i64 %i 
  %1 = load i64, i64* %0 
  ret i64 %1 
} 
load_shift: 
 ldr x0, [x0, x1, lsl #3] 
 ret

## 第 80 頁

INSTRUCTION SELECTION5/5
%struct.A = type { i64*, [16 x i64] } 
define i64 @get_val(%struct.A* %p, i64 %i, i64 %j) { 
ent: 
  %0 = getelementptr %struct.A, %struct.A* %p, i64 %i, i32 1, i64 %j 
  %1 = load i64, i64* %0 
  ret i64 %1 
}
get_val: 
 movz w8, #0x88 
 madd x8, x1, x8, x0 
 add x8, x8, x2, lsl #3 
 ldr x0, [x8, #8] 
 ret

## 第 81 頁

SSA DESTRUCTION1/5
How do we deal with the phi functions?
Copy the assignment operator to the end of the predecessor.
(Must be done carefully)

## 第 82 頁

SSA DESTRUCTION2/3
cmp = icmp eq, cond, 0
br cmp, B1, B2
B 0
b r  B 3
B 1
b r  B 3
B 2
a = phi (3, B1) (7, B2)
print(a)
B 3
cmp = icmp eq, cond, 0
br cmp, B1, B2
B 0
a  =  3
b r  B 3
B 1
a  =  7
b r  B 3
B 2
print(a)
B 3
converts to
Example to show the assignment copying

## 第 83 頁

SSA DESTRUCTION3/5
i.0 = phi (0, B0) (i.1, B2)
cmp = icmp lt, i.0, 100
br cmp, B2, B3
B 1
ret
B 3
print(i.0)
i.1 = add i.0, 1
br B1
B 2
b r  B 1
B 0
cmp = icmp lt, i.0, 100
br cmp, B2, B3
B 1 B 3
print(i.0)
i.1 = add i.0, 1
i.0 = i.1
br B1
B 2
i.0 = 0
br B1
B 0
ret
Converts to
Converts to
Example to show the assignment copying with loop

## 第 84 頁

SSA DESTRUCTION4/5
i.0 = phi (0, B0) (i.1, B2)
cmp = icmp lt, i.0, 100
br cmp, B2, B3
B 1
print(i.0)
ret
B 3
print(i.0)
i.1 = add i.0, 1
br B1
B 2
b r  B 1
B 0
cmp = icmp lt, i.0, 100
br cmp, B2, B3
B 1 B 3
print(i.0)
i.1 = add i.0, 1
i.0 = i.1
br B1
B 2
i.0 = 0
br B1
B 0
print(i.0)
ret
Doesn't
work!
Lost copy problem — Naive de-SSA algorithm doesn't work
due to live range conflicts. Need extra assignments and
renaming to avoid conflicts.

## 第 85 頁

SSA DESTRUCTION5/5
t = x
x = y
y = t
br cmp, B2, B1
B 1
print(x)
print(y)
B 2
x = 1
y = 2
br B1
B 0
x1 = phi (x0, y1)
y1 = phi (y0, x1)
br cmp, B2, B1
B 1
print(x1)
print(y1)
B 2
x0 = 1
y0 = 2
br B1
B 0
x1 = y1
y1 = x1
br cmp, B2, B1
B 1
print(x1)
print(y1)
B 2
x0 = 1
y0 = 2
x1 = x0
y1= y0
br B1
B 0
Swap problem — Conflicts due to parallel assignment
semantics of phi instructions. A correct algorithm should
detect the cycle and implement parallel assignment with
swap instructions.

## 第 86 頁

REGISTER ALLOCATION1/2
We have to replace infinite virtual registers with finite
machine registers.
Register Allocation — Make sure that the maximum
simultaneous register usages are less then k.
Register Assignment — Assign a register given the fact that
registers are always suﬀicient.
The classical solution is to compute the life time of each
variables, build the inference graph, spill the variable to
stack if the inference graph is not k-colorable.

## 第 87 頁

REGISTER ALLOCATION2/2
r e t  y
a d d  y ,  c ,  g
a d d  g  a  f
a d d  f ,  d ,  e
a d d  e ,  a ,  c
a d d  d ,  a ,  b
m o v  c ,  # 2
m o v  b ,  # 1
m o v  a ,  # 1 a
b
c
d
e
f
g
y
a
g
b
c
e
d
f
y

## 第 88 頁

INSTRUCTION SCHEDULING1/2
Sort the instruction according the number of cycles in order
to reduce execution time of the critical path.
Constraints: (a) Data dependence, (b) Functional units, (c)
Issue width, (d) Datapath forwarding, (e) Instruction cycle
time, etc.

## 第 89 頁

INSTRUCTION SCHEDULING2/2
0: add r0, r0, r1 
1: add r1, r2, r2 
2: ldr r4, [r3] 
3: sub r4, r4, r0 
4: str r4, [r1]
2: ldr r4, [r3]  # load instruction needs more cycles 
0: add r0, r0, r1 
1: add r1, r2, r2 
3: sub r4, r4, r0 
4: str r4, [r1]

## 第 90 頁

COMPILER AND ICT
INDUSTRY

## 第 91 頁

WHERE CAN WE FIND COMPILER?

## 第 92 頁

SMART PHONES
Android ART (Java VM)
OpenGL|ES GLSL Compiler (Graphics)

## 第 93 頁

BROWSERS
Javascript-related
Javascript Engine, e.g. V8, IonMonkey, JavaScriptCore,
Chakra
Regular Expression Engine
WebAssembly
WebGL — OpenGL binding for the Web (Graphics)
WebCL — OpenCL binding for the Web (Computing)

## 第 94 頁

DESKTOP APPLICATION RUN-TIME
ENVIRONMENTS
Java run-time (Java, Scala, Clojure, Groovy)
.NET run-time (C#, F#)
Ruby
Python
PHP

## 第 95 頁

VIRTUAL MACHINES AND EMULATORS
Simulate diﬀerent architecture (using dynamic
recompilation), e.g. QEMU
Simulate special instructions that are not supported by
hypervisor.

## 第 96 頁

DEVELOPMENT ENVIRONMENT
C/C++ compiler, e.g. MSVC, GCC, LLVM
Java compiler, e.g. OpenJDK
DSP compilers for digital signal processors.
VHDL compilers for IC design team.
Profiling and/or intrusmentation tools, e.g. Valgrind, Dr.
Memory, Address Sanitizer, Memory Sanitizer

## 第 97 頁

WHAT DOES A COMPILER TEAM DO?
Develop the compiler toolchain for the in-house processors,
e.g. DSP, GPU, CPU, and etc.
Tune the performance of the existing compiler or virtual
machines.
Develop a new programming language or model (e.g. Cuda
from NVIDIA.)

## 第 98 頁

SOME RESEARCH TOPICS ...
Memory model for concurrency — Designing a good
memory model at programming language level, which
should be intuitive to human while open for
compiler/architecture optimization, is still a challenging
problem.
Both Java and C++ took eﬀorts to standardize. However,
there are still some unintuitive cases that are allowed and
some desirable cases being ruled out.

## 第 99 頁

THE END
Q & A
