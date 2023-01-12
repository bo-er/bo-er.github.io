---
layout: post
title: goto can be helpful
gh-repo: bo-er/bo-er.github.io
gh-badge: [star, fork, follow]
tags: [GO,goto,panic-recover]
comments: true
---

In a 1968 paper, Dijkstra wrote 'GOTO Considered Harmful,' arguing that a "goto" would generally decrease code readability and undermine the program's maintainability. His argument is based on common sense: if the code is hard to read, it must also be hard to debug. In other words, you have to understand the code before you can make any modifications to it.

But if a "goto" can be so harmful, why was it introduced into the C language and so many other computer languages of the C family? This is presumably because "goto" was invented and heavily used in assembly language in an era without a high-level programming language. And by "harmful," Dijkstra didn't oppose the existence of "goto." What he fought against was the abusive use of "goto" in the 80s, which led to the "spaghetti code" of FORTRAN programs.

## Assembly code loves goto

Recently, I have been reading a book named "Computer Science: A Programmer's Perspective." In this book, the assembly language version of "goto": "jmp," is not considered "harmful" but rather "useful."  
There are two approaches to transfer execution in assembly: jump and branching, the former is used for transferring execution unconditionally, and the latter is used for transferring execution conditionally.
The "goto" in modern computer language is a legacy of assembly languages from the earliest days of programming.

Now, You might wonder where "goto"'s place is in assembly. Here is an example of a simple if-else C code:

```C  
if (test-expr)  
    then-statement  
else  
    else-statement  
```

It can surprise many people that its assembly implementation looks like the following code, with a structure that is quite different from the ubiquitous if-else we see in almost every programming language.  
```C  
t = test-expr  
if (!t)  
    goto false  // conditional branching here  
then-statement  
goto done   // unconditional branching here  
false:  
  else-statement  
done:    
```

We can see that "goto" is useful in assembly code. They're even more useful when you add a condition, the assembly code contains both a conditional branch and an unconditional branch  to ensure the correct branch executes. Here are some frequently used conditional jump instructions in assembly code:

| Instruction | Description | Jump condition |  
| ----------- | ----------- | ------  |  
| jmp Operadn | Indirect jump |1|  
|je Label|Equal/Zero|ZF|  
|jne Label|not Equal/not Zero|~ZF|  
|js Label|Negative|SF|  
|jns Label|NonNegative|~SF|  
|jg Label|Greater|~(SF^OF) & ZF|  
|jl Label|Less|SF^OF|  
|...|...|...|

Loops is another example illustrates why goto is important.

(1) In C, we may write a loop like this:  
```C  
long add_to_one(long n)  
{  
    long result = 0;  
    do{  
        result +=n;  
        n--;  
    } while(n>0);  
    return result;  
}  
```

(2) The equivalent goto version is:  
```C  
long add_to_one(long n)  
{  
        long result = 0;  
    loop:  
        result +=n;  
        n--;  
        if (n > 0)  
            goto loop;  
        return result;  
}

```

(3). The corresponding assembly version is:  
```  
long add_to_one(long n)  
n in %rdi  
add_to_one:  
    movl $0 %eax //Set result to 0  
.L2:  
    addq %rdi, %rax // Compute result +=n  
    subq $1, %rdi   // Decrement n by 1  
    cmpq $0, %rd1   // Compare n to 0  
    jg   .L2        // if n > 0, goto loop  
    rep; ret        // return  
```

  
## Where "goto" fits in modern languages

Do computers hate "goto"? Of course not! As explained at the beginning of this article, "goto" is considered harmful because it undermines the readability and maintainability of the program when misused.  
We have seen that "goto" can be helpful in assembly language by giving examples like loops and if-else statements. However, for high-level programming, we shouldn't be using them in these scenarios since "goto" will not improve but on the contray undermine readability and maintainability in such cases.  
Where goto still fits in high-level programming languages is error handling, especially for languages lacking try-catch mechanisms like C and GO.     

On Wikipedia's "Goto" page, I found content that share similar view- from the author of _The C Programming Language_ :

>In The C Programming Language, Brian Kernighan and Dennis Ritchie warn that goto is "infinitely abusable" but also suggest that it could be used for end-of-function error handlers and multi-level breaks from loops. These two patterns can be found in numerous subsequent books on C by other authors; a 2007 introductory textbook notes that the error handling pattern is a way to work around the "lack of built-in exception handling within the C language.

  
### End-of-function error handling  
Here I am writing a GO code to prove the usage of goto for end-of-function error handling, and it can be beneficial when you possibly have to handle a row of errors.  
```GO  
func goodHandling() error {

var err error

if err = A(); err != nil {  
    goto Err  
}  
if err = B(); err != nil {  
    goto Err  
}  
if err = C(); err != nil {  
    goto Err  
}  
if err = D(); err != nil {  
    goto Err  
}

Err:  
    if PrintLogOnly(err) {  
        log.Println("an error occurs during processing: ", err.Error())  
        return nil  
    }  
    return err  
}

func badHandling() error {

var err error

if err = A(); err != nil {  
    if PrintLogOnly(err) {  
        log.Println("an error occurs during processing: ", err.Error())  
        return nil  
    }  
    return err  
    }  
if err = B(); err != nil {  
    if PrintLogOnly(err) {  
        log.Println("an error occurs during processing: ", err.Error())  
        return nil  
    }  
    return err  
    }  
if err = C(); err != nil {  
    if PrintLogOnly(err) {  
        log.Println("an error occurs during processing: ", err.Error())  
        return nil  
    }  
    return err  
    }  
if err = D(); err != nil {  
    if PrintLogOnly(err) {  
        log.Println("an error occurs during processing: ", err.Error())  
        return nil  
    }  
    return err  
    }  
}

    
func PrintLogOnly(e error) bool {  
    return false  
}

func A() error {  
    return nil  
}

func B() error {  
    return nil  
}

func C() error {  
    return nil  
}

func D() error {  
    return nil  
}  
```

It's evident that the combination of "goto" and the label "Err" makes the error-handling code nicer and cleaner. Our goal of improving code readability and maintainability has been achieved.

### Breaking multi-level loops
Here is a GO example illustrating this:  
```GO

func main(){  
    for _,i := range s1{  
        for _,j := range i{  
            for _,k := range j{  
                goto done  
            }  
        }  
    }  
    done:  
        return  
}

```

The above examples are simple and can be categorized as "local goto", thery're local since a typical "goto" is defined and used within a single function block. Besides local goto there is another concept called non-local goto.

## Panic and recover: a new nonlocal magic

Panic and recover are features of GO; you might be wondering why their combination is also a type of "goto". This is because, in essence, "goto" is an unconditional branch-taking keyword. The behavior of panic and recover in GO is analogous to "goto": the code panics in one place and "jumps" to the recover part of the code. The panic and recover in GO work beyond the "local goto" discussed above given their ability to break function boundary. 

Below is a sample GO code illustrating the "non-local goto" . I also call it the GO implementation of try-catch error handling:

```GO  
func catch() {  
    if v := recover(); v != nil {  
        log.Println("captured someone panicking:", v)  
    }  
}

func throw(err error) {  
    panic(err)  
}

func main() {  
    defer catch()  
    try()  
}

func try() {  
    if err:= doSomething();err != nil{  
        throw(errors.New("he panics a lot"))  
    }  
}

```

The above code can be beneficial when working with nested function calls. Error handling in such a situation can be very tedious, especially when you don't expect the code to go wrong, and this is precisely why try-catch style error handling was born: to release the programmer from error checking and enable them to spend more time on program logic that really matters.

In C, the "nonlocal goto" is implemented by the combination of two library functions, "setjmp" and "longjmp", which enable C code to "goto" a location outside of the currently executing function. These two library functions work together as a useful approach to error handling:

```C  
#include <stdio.h>  
#include <setjmp.h>

#define TRY do{ jmp_buf ex_buf__; if( !setjmp(ex_buf__) ){  
#define CATCH } else {  
#define ETRY } }while(0)  
#define THROW longjmp(ex_buf__, 1)

int  
main(int argc, char** argv)  
{  
   TRY  
   {  
      printf("In Try Statement\n");  
      THROW;  
      printf("I do not appear\n");  
   }  
   CATCH  
   {  
      printf("Got Exception!\n");  
   }  
   ETRY;

   return 0;  
}  
```

Why does C not support using just a simple"goto" to jump to another function? This is primarily because in C, there is no such thing as "nested function," and a C programmer can't define a function inside of another. As a result, the compiler can't be sure when function A that calls a "goto L1" runs, whether another function B that contains the label "L1" is also on the stack frame.  If function B is not on the stack and execution jumps from A to B, the variables in function B may not have been pushed into the stack when referenced.

```C
int B(){
    int a = 10;
    L1:
        printf("a is: %d",a);
}

int A(){
    goto L1;
}
```

When it comes to GO, the language does allow nested functions, but however, let alone jumping across functions, GO even doesn't allow "goto" its outer/inner function as explained in the GO Specification:
>"The scope of a label is the body of the function, in which it is declared and excludes the body of any nested function."

If you run this piece of GO code, the compiler fails with: label Greeting not defined
```GO
func hello() {
	g := func() {
		goto Greeting // compiler:label Greeting not defined
	}
	{
		goto Greeting
	}
	g()

Greeting:
	fmt.Println("How are you")

}
```

The reason behind this design decision is not given by the GO doc but can be guessed. First, there are no compelling reasons for the GO team to implement the feature. Second, by supporting it, the compiler would be more complex than it is now.

### The standard GO library uses nonlocal goto

Nonlocal goto can help handle unexpected situations. For example, in  Go standard library "json", encodeState's method marshal recovers from a jsonError panic, this panic is not intended to stop the program, its usage is to break out of a nested loop of fields marshaling.

```GO
      // jsonError is an error wrapper type for internal use only.
     // Panics with errors are wrapped in jsonError so that the top-level recover
     // can distinguish intentional panics from this package.
     type jsonError struct{ error }
     
     func (e *encodeState) marshal(v any, opts encOpts) (err error) {
     	defer func() {
     		if r := recover(); r != nil {
     			if je, ok := r.(jsonError); ok {
     				err = je.error
     			} else {
     				panic(r)
     			}
     		}
     	}()
     	e.reflectValue(reflect.ValueOf(v), opts)
     	return nil
     }
```

### try-catch across goroutines is not possible

In GO, Calling t.Fatal from a non-test goroutine is not advised as it may produce unexpected behavior.
There is an example that fails to pass the `go vet` command:
```GO
func TestBadPanic(t *testing.T) {

    go func() {
        err := doSomething()
        t.Fatal(err)
    }()
}
```

You may wonder if we could bypass this with the "nonlocal goto" magic. What would happen when you run the below code?
Would the panic be recovered? 

```GO
type testError struct{ error }  
  
func throwTestError(e error) {  
   panic(testError{e})  
}  
  
func catchError(t *testing.T) {  
   if r := recover(); r != nil {  
      if te, ok := r.(testError); ok {  
         t.Fatal(te)  
      } else {  
         panic(r)  
      }  
   }  
}

// the panic will not be recovered.
func Test_PanicRecoverAcrossGoroutines(t *testing.T) {
	defer catchError(t)
    go func() {
        err := doSomething()
        throwTestError(err)
    }()
}

```

The answer is a sad no. This behavior is documented in [# The Go Programming Language Specification](https://go.dev/ref/spec#Handling_panics)

>The return value of `recover` is `nil` if any of the following conditions hold:

-   `panic`'s argument was `nil`;
-   the goroutine is not panicking;
-   `recover` was not called directly by a deferred function.

Since the goroutine of the test function, Test_PanicRecoverAcrossGoroutines is not panicking, the deferred function catchError has no affect on the panic.

The restriction that a panic can't be recovered from another goroutine is explained in [Effective GO](https://go.dev/doc/effective_go#panic). A panic immediately stops the execution of the current function and begins unwinding the stack of the goroutine; unwinding means stack unwinding, it's a process of removing function entries from the stack at runtime, running any deferred functions along the way. This is why we must recover in a deferred function, no function may run at the unwinding stage except the deferred one. If that unwinding reaches the top of the goroutine's stack, the program dies. Therefore if we try to recover the panic outside of the scope of the goroutine, the effort is in vain; the deferred recover function outside of the goroutine's scope wouldn't be pushed into the goroutine's stack frame at its unwinding process.

## Conclusion
Anyway, the main takeaway is that goto is not harmful when you use it in the right place. If you're an assembly programmer, you must use "goto" daily. If you use high-level programming languages like GO, you can use "goto" to improve your code readability and maintainability in places like error handling. This article also gives an example of using panic and recover together as a non-local goto, just like the combination of "setjmp()" and "longjmp()" library functions in C. With the help of panic and recover, magic happens: a GO version of the try-catch mechanism is born. Of course, it has limitations, like you can't throw an error in one goroutine and try to catch it in another.  