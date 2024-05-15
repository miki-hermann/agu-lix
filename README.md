# THIS README PAGE IS NOT OPERATIONAL YET

# PRIMAL GRAMMARS DRIVEN AUTOMATED INDUCTION

## Table of contents

* [Binaries](#binaries)
* [Software](#software)
* [Examples](#examples)
* [Syntax of the specifications](#syntax-of-the-specifications)
* [Form of the output](#form-of-the-output)

## Binaries

## Software

The name  of the software  is *spec* (for  Linux and macOS)  or *spec.exe*
(for Windows). It can be called by a command line of the long form

```bash
spec --input <input file> --output <output file> --trace <number> --rounds <number> --direction <direction>
```
or of the short form

```bash
spec -i <input file> -o <output file> -t <number> -r <number> -d <direction>
```
You can combine flags of both forms. Neither of the parameters need to
be  specified,  since  all  have  a default  for  missing  flags.  The
respective defaults are:

   - -i STDIN
   - -o STDOUT
   - -t 3
   - -r 10
   - -d first

The flag  -t indicates how many  clauses are necessary to  construct a
primal grammar (see Proposition 9 in the paper). The -d flag indicates
which induction variables should be  taken : either `first` or `last`.
The -r flag indicates the maximal number of rounds of proof, i.e., how
many times  the INDUCTIVE NARROWING rule  can be called. If  the proof
does  not terminate  with SUCCESS  or  DISPROOF when  it reaches  this
limit, a

	loop limit reached

warning is issued and the proof is cancelled.

## Examples

The **examples** directory  contains 56 different specifications with
conjectures to prove. All input files are of the form

	<number>-<short description>-input.txt

The runs of the corresponding specifications can be found in the files

	<number>-<short description>-output.txt

This directory also contains a Makefile,  which can be used to run the
software on  the specifications.  To run all  examples, just  type the
command

```Makefile
make
```
If you want to run only one example, just type the command

```Makefile
make <number>
```
where `<number>` corresponds to the leading prefix of the input file. If
you want to  run the examples in the interval  from `<first>` to `<last>`,
type the command

```bash
run-examples -i <first> <last>
```
where `<first>` and `<last>` are prefix numbers of the input files. If you
want to run more examples at once, type the command

```bash
run-examples <number>+
```
with prefix numbers of the specifications. The numbers need not be in any order.

Once you have run your desired examples, you can print a statistics of
the runs by typing the command

```bash
check-examples
```
To run  the commands *run-examples*  and *check-examples* you  need to
have *perl* installed on your computer.

## Syntax of the specifications

The syntax of input specifications is described by the following LL(1)
context-free grammar in BNF using regular expressions.  Curly brackets
are used for  grouping, the bar means an alternative,  * is the Kleene
star for iteration, + means at least one occurrence, the question mark
?  means 0 or 1 occurrence(s) of the preceding item.  The non-terminal
symbols are enclosed in <...>, terminal symbols are in quotation marks
'...'. The staring non-terminal symbol is `<specification>`.

	<specification> ::= <spec> <include>? <sorts> <cons> <defs> <prec> <axioms> <cjts>
	<spec>          ::= 'specification' <ident>
	<include>       ::= 'include' <inclusion>+
	<inclusion>     ::= '<' <ident> '>' | '"' <ident> '"'
	<sorts>         ::= 'sorts' <ident>+
	<cons>          ::= 'constructors' <fundef>+
	<defs>          ::= 'defined' 'functions'? <fundef>+
	<prec>          ::= 'precedence' <symb> {{'>' | '='} <symb>}+
	<axioms>        ::= 'axioms' {<cond> ';'}+
	<cjts>          ::= 'conjectures' {<cond> ';'}+
	<fundef>        ::= <symb> {',' <symb>}* ':' <ident>* '->' <ident>
	<symb>          ::= <ident> | <number> | <cmpop> | <addop> | <multop>

	<cond>    ::= <eq> { {'&' | '/\'} <eq>}* {'=>' <eq>}?
	<eq>      ::= <comp> '=' <comp>
	<comp>    ::= <exp> <compop> <exp> | <exp>
	<cmpop>   ::= '==' | '!=' | '>' | '>=' | '<' | '<='
	<exp>     ::= <term> <addop> <exp> | <term>
	<addop>   ::= '+' | '-' | '||' | ':' | '@@'
	<term>    ::= <factor> <multop> <term> | <factor>
	<multop>  ::= '*' | '/' | '%' | '&&' | '..'
	<factor>  ::= '(' <comp> ')' | '!' <factor> | <funct> | <funct> '(' <comp> {',' <comp>}* ')'
	<funct>   ::= <ident> | <number>
	<ident>   ::= [a-zA-Z]+
	<number>  ::= [0-9]+

Every identifier starting with a  character from the set {u,v,w,x,y,z}
or {U,V,W,X,Y,Z} is  a variable. Note neither the  underscore "_", nor
the digits can be used  in the identifiers.  Identifiers `<ident>` are
exclusively composed  of small  and capital letter  from "a"  to "z",
resp. from "A" to "Z".

The  binary  operators  for   comparison  (`<compop>`),  for  addition
(`<addop>`),  and  for  multiplications (`<multop>`)  have  the  infix
notation with the usual precedence.

Equality in  clauses is denoted  by "=", not by  "==" which is  just a
comparison  operator.   Conditional  clauses   use  "=>"  to  separate
conditions on the left from  equational clause on the right. Equations
in the condition are regrouped by the operator "&" or "/\".

Precedence  must   be  declared  for  all   constructors  and  defined
symbols. The precedence is one non-increasing chain of symbols ordered
by greater-than  (>) and equality  (=) operators. Since there  is only
one non-increasing chain, all symbols are comparable.

Specification   modules  can   be  hierarchically   included  by   the
`<include>`  construction.  When  the identifier  is enclosed  in <..>
then it  is loaded from the  standard library. When the  identifier is
enclosed in  quotation marks ".."   then it  is loaded from  the local
directory. Neither the precedence on symbols, nor the conjectures, are
inherited by inclusion.

## Form of the output

The form of the output files follows some specific rules.

Every identifier starting with a  character from the set {u,v,w,x,y,z}
or  {U,V,W,X,Y,Z}  is a  variable.  Defined  symbols start  with  "^".
Counter variables start with "$". Rolling Variables terminate with "@"
or have a counter suffix.

Hence, if you see the rewrite rules

       ^f0(0; y@) -> nil
       ^f0($n0 + 1; y@) -> cons(y$n0, ^f0($n0; y@))

then their semantics is:

   - *^f0* is a defined symbol,
   - *$n0* is a counter variable,
   - *y@* is a rolling variable,
   - *y$n0* is the rolling variable y with index equal to the value of *$n0*,
   - *nil* and *cons* are identifiers from the specification.

EOF
