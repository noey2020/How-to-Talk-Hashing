# How-to-Talk-Hashing

September 19, 2019

Hashing

I appreciate comments. Shoot me an email at noel_s_cruz@yahoo.com!

Hire me!

Search operations are common activities in computing. This is why we continually
study and innovate for new effective methods of search. The search tree ADT
allowed various operations on a set of elements. We discuss the hash table ADT
which supports only a subset of the operations allowed by binary search trees.

The implementation of hash tables is frequently called hashing. Hashing is a 
technique used for performing insertions, deletions, and finds in constant 
average time. Tree operations that require any ordering information among the
elements are not supported efficiently. The central data structure hence, is 
the hash table.

General Idea

The ideal hash table data structure is merely an array of some fixed size
containing the items. Generally a search is performed on some part (that is,
data member) of the item. This is called the key. For instance, an item could
consist of a string (that serves as the key) and additional data members (for 
instance, a name that is part of a large employee structure). We will refer to 
the table size as TableSize, with the understanding that this is part of a hash
data structure and not merely some variable floating around globally. The 
common convention is to have the table run from 0 to TableSize - 1; we will see
why shortly.

Each key is mapped into some number in the range 0 to TableSize - 1 and placed
in the appropriate cell. The mapping is called a hash function, which ideally 
should be simple to compute and should ensure that any two distinct keys get 
different cells. Since there are a finite number of cells and a virtually 
inexhaustible supply of keys, this is clearly impossible, and thus we seek a
hash function that distributes the keys evenly among the cells. Figure 1 is 
typical of a perfect situation. In this example, john hashes to 3, phil hashes
to 4, dave hashes to 6, and mary hashes to 7.

Figure 1 An Ideal Hash Table

0

1

2

3   john   25000

4   phil   31250

5

6   dave   27500

7   mary   28200

8

9

This is the basic idea of hashing. The only remaining problems deal with 
choosing a function, deciding what to do when two keys hash to the same value
(this is known as a collision), and deciding on the table size.

Hash Function

If the input keys are integers, then simply returning Key modulus TableSize is
generally a reasonable strategy, unless Key happens to have some undesirable
properties. In this case, the choice of hash function needs to be carefully
considered. For instance, if the table size is 10 and the keys all end in zero,
then the standard hash function is a bad choice. For reasons we shall see later,
and to avoid situations like the one above, it is often a good idea to ensure
that the table size is prime. When the input keys are random integers, then 
this function is not only very simple to compute but also distributes the keys
evenly.

Usually the keys are strings; in this case, the hash function needs to be 
chosen carefully. One option is to add up the ASCII values of the characters in
the string. The routine in Figure 2 implements this strategy.

Figure 2 A Simple Hash Function

int hash( const string & key, int tableSize )

{

  int hashVal = 0;
  
  for ( char ch : key )
  
    hashVal += ch;
    
  return hashVal % tableSize;
  
}

The hash function depicted in Figure 2 is simple to implement and computes an
answer quickly. However, if the table size is large, the function does not
distribute the keys well. For instance, suppose that TableSize = 10,007 (10,007
is a prime number). Suppose all the keys are eight or fewer characters long.
Since an ASCII character has an integer value that is always at most 127, the
hash function typically can only assume values between 0 and 1,016, which is
127*8. This is clearly not an equitable distribution!

September 22, 2019

Figure 4 shows a third attempt at a hash function. This hash function involves
all characters in the key and can generally be expected to distribute well (it
computes summation from i=0 to KeySize-1 Key[KeySize-1 -1].37**i and brings the
result into proper range). The code computes a polynomial function (of 37) by
use of Horner's rule. For instance, another way of computing 
hofk = k0+37k1+37**2k2 is by the formula hofk = ((k2)*37+k1)*37+k0. Horner's 
rule extends this to an nth degree polynomial.

The hash function takes advantage of the fact that overflow is allowed and uses
unsigned int to avoid introducing a negative number.

Figure 4 A Good Hash function

/**
 * A hash routine for string objects.
 */

unsigned int hash( const string & key, int tableSize )

{

  unsigned int hashVal = 0;
  
  for( char ch : key )
  
    hashVal = 37 * hashVal + ch;
    
  return hashVal % tableSize;
  
}

The main programming detail left is collision resolution. If, when an element is
inserted, it hashes to the same value as an already inserted element, then we 
have a collision and need to resolve it. There are several methods for dealing
with this. We will discuss two of the simplest: separate chaining and open
addressing; then we will look at some more recently discovered alternatives.

Separate Chaining

The first strategy, commonly known as separate chaining, is to keep a list of
all elements that hash to the same value. We can use the Standard Library list 
implementation.

june 11, 2018

Hashing

In the preceding two sections we assumed that the record being sought is stored
in a table and that it is necessary to pass through some number of keys before
finding the desired one. The organization of the file (sequential, indexed
sequential, binary tree, and so forth) and the order in which the keys are 
inserted affect the number of keys that must be inspected before obtaining the
desired one. Obviously, the efficient search techniques are those that minimize
the number of these comparisons. Optimally, we would like to have a table 
organization and search technique in which there are no unnecessary comparisons.
Let us see if this is feasible.

If each key is to be retrieved in a single access, the location of the record
within the table can depend only on the key; it may not depend on the locations
of other keys as in a tree. The most efficient way to organize such a table is 
an array (that is, each record is stored at a specific offset from the base 
address of the table). If the record keys are integers, the keys themselves can
serve themselves as indices to the array.

Let us consider an example of such a system. Suppose a manufacturing company 
has an inventory file consisting of 100 parts, each part having a unique
two-digit part number. Then the obvious way to store this file is to declare an
array

parttype part[100];

where part[i] represents the record whose part number is i. In this situation,
the part numbers are keys that are used as indices to the array. Even if the
company stocks fewer than 100 parts, the same structure can be used to maintain
the inventory file. Although many locations in part may correspond to 
nonexistent keys, this waste is offset by the advantage of direct access to
each of the existent parts.

Unfortunately, however,such a system is not always practical. For example, 
suppose that the company has an inventory file of more than 100 items and the
key to each record is a seven-digit part number. To use direct indexing using 
the entire seven-digit key, an array of 10 million elements would be required.
This clearly wastes an unacceptably large amount of space because it is 
extremely unlikely that a company stocks more than a few thousand parts.

What is necessary is some method of converting a key into an integer within a
limited range. Ideally, no two keys should be converted into the same integer.
Unfortunately, such an ideal method usually does not exist. Let us attempt to
develop methods that come close to the ideal, and determine what action to take
when the ideal is achieved.

Let us reconsider the example of a company with an inventory file in which each
record is keyed by a seven-digit part number. Suppose that the company has 
fewer than 1000 parts and that there is ony a single record for each part. Then
an array of 1000 elements is sufficient to contain the entire file. The array is
indexed by an integer between 0 and 999 inclusive. The last three digits of the 
part number are used to as the index for the part's record in the array. Note 
that two keys that are relatively close to each other numerically, such as
4618396 and 4618996 may be farther from each other in the table than two keys
that are widely separated numerically, such as 0000991 and 9846995. Only the 
last three digits of the key are used in determining the position of a record.

A function that transforms a key into a table index is called a hash function. 
if h is a hash function and key is a key, h(key) is called the hash of key and 
is the index at which a record with the key key should be placed. If r is a 
record whose key hashes into hr, hr is called the hash key of r. 

##############

The hash function in the preceding example is h(k) = key % 1000.

##############

The values that h produces should cover the entire set of indices in the table.
For example, tje function x % 1000 can produce any integer between 0 and 999,
depending on the value of x. As we will see shortly, is is a good idea for the
table size to be somewhat larger than the number of records that are to be 
inserted. 

The foregoing method has one flaw. Suppose that two keys k1 and k2 are such 
that h(k1) equals h(k2). Then when a record with key k1 is entered into the 
table, it is inserted at position h(k1). Then when k2 is hashed, because its 
hash key is the same as k1, an attempt may be made to insert the record into 
the same position where the record with key k1 is stored. Clearly, two records
cannot occupy the same position . Such a situation is called a hash collisin or
a hash clash. 

There are two basic methods of dealing with a hash clash.


##############
june 13, 2018

https://algs4.cs.princeton.edu/home/

Algorithms, 4th Edition by Robert Sedgewick and Kevin Wayne surveys the most 
important algorithms and data structures in use today.

essential information that
every serious programmer
needs to know about
algorithms and data structures 

The textbook is organized into six chapters: 
Chapter 1: Fundamentals introduces a scientific and engineering basis for
  comparing algorithms and making predictions. It also includes our 
  programming model. 
Chapter 2: Sorting considers several classic sorting algorithms, including 
  insertion sort, mergesort, and quicksort. It also features a binary heap 
  implementation of a priority queue. 
Chapter 3: Searching describes several classic symbol-table implementations,
  including binary search trees, red–black trees, and hash tables. 
Chapter 4: Graphs surveys the most important graph-processing problems, 
  including depth-first search, breadth-first search, minimum spanning trees,
  and shortest paths. 
Chapter 5: Strings investigates specialized algorithms for string processing,
  including radix sorting, substring search, tries, regular expressions, and 
  data compression. 
Chapter 6: Context highlights connections to systems programming, scientific
  computing, commercial applications, operations research, and intractability. 

If keys are small integers, we can use an array to implement a symbol table, by
interpreting the key as an array index so that we can store the value
associated with key i in array position i. In this section, we consider hashing,
an extension of this simple method that handles more complicated types of keys.
We reference key-value pairs using arrays by doing arithmetic operations to 
transform keys into array indices. 

Search algorithms that use hashing consist of two separate parts. The first 
step is to compute a hash function that transforms the search key into an array
index. Ideally, different keys would map to different indices. This ideal is
generally beyond our reach, so we have to face the possibility that two or more 
different keys may hash to the same array index. Thus, the second part of a 
hashing search is a collision-resolution process that deals with this situation. 

Hash functions. If we have an array that can hold M key-value pairs, then we
need a function that can transform any given key into an index into that array:
an integer in the range [0, M-1]. We seek a hash function that is both easy to
compute and uniformly distributes the keys. 

  - Typical example. Suppose that we have an application where the keys are U.S.
    social security numbers. A social security number such as 123-45-6789 is a 
    9-digit number divided into three fields. The first field identifies the 
    geographical area where the number was issued (for example number whose 
    first field are 035 are from Rhode Island and numbers whose first field are
    214 are from Maryland) and the other two fields identify the individual. 
    There are a billion different social security numbers, but suppose that our
    application will need to process just a few hundred keys, so that we could 
    use a hash table of size M = 1000. One possible approach to implementing a
    hash function is to use three digits from the key. Using three digits from 
    the field on the right is likely to be preferable to using the three digits 
    in the field on the left (since customers may not be equally dispersed over
    geographic areas), but a better approach is to use all nine digits to make 
    an int value, then consider hash functions for integers, described next. 

  - Positive integers. The most commonly used method for hashing integers is 
    called modular hashing: we choose the array size M to be prime, and, for
    any positive integer key k, compute the remainder when dividing k by M. 
    This function is very easy to compute (k % M, in Java), and is effective in
    dispersing the keys evenly between 0 and M-1. 

  - Floating-point numbers. If the keys are real numbers between 0 and 1, we
    might just multiply by M and round off to the nearest integer to get an 
    index between 0 and M-1. Although it is intuitive, this approach is 
    defective because it gives more weight to the most significant bits of the
    keys; the least significant bits play no role. One way to address this 
    situation is to use modular hashing on the binary representation of the key
    (this is what Java does). 

  - Strings. Modular hashing works for long keys such as strings, too: we 
    simply treat them as huge integers. For example, the code below computes a 
    modular hash function for a String s, where R is a small prime integer 
    (Java uses 31). 
    
      int hash = 0;
      
      for (int i = 0; i < s.length(); i++)
      
      hash = (R * hash + s.charAt(i)) % M;

  - Compound keys. If the key type has multiple integer fields, we can 
    typically mix them together in the way just described for String values.
    For example, suppose that search keys are of type USPhoneNumber.java, which
    has three integer fields area (3-digit area code), exch (3-digit exchange),
    and ext (4-digit extension). In this case, we can compute the number 
    
      int hash = (((area * R + exch) % M) * R + ext) % M; 

  - Java conventions. Java helps us address the basic problem that every type
    of data needs a hash function by requiring that every data type must 
    implement a method called hashCode() (which returns a 32-bit integer). The
    implementation of hashCode() for an object must be consistent with equals.
    That is, if a.equals(b) is true, then a.hashCode() must have the same
    numerical value as b.hashCode(). If the hashCode() values are the same, 
    the objects may or may not be equal, and we must use equals() to decide
    which condition holds. 

  - Converting a hashCode() to an array index. Since our goal is an array 
    index, not a 32-bit integer, we combine hashCode() with modular hashing in
    our implementations to produce integers between 0 and M-1 as follows: 
    
      private int hash(Key key) {
      
        return (key.hashCode() & 0x7fffffff) % M;
        
      }

    The code masks off the sign bit (to turn the 32-bit integer into a 31-bit
    nonnegative integer) and then computing the remainder when dividing by M,
    as in modular hashing. 

  - User-defined hashCode(). Client code expects that hashCode() disperses the
    keys uniformly among the possible 32-bit result values. That is, for any 
    object x, you can write x.hashCode() and, in principle, expect to get any 
    one of the 2^32 possible 32-bit values with equal likelihood. Java provides
    hashCode() implementations that aspire to this functionality for many
    common types (including String, Integer, Double, Date, and URL), but for 
    your own type, you have to try to do it on your own. Program 
    PhoneNumber.java illustrates one way to proceed: make integers from the 
    instance variables and use modular hashing. Program Transaction.java
    illustrates an even simpler approach: use the hashCode() method for the
    instance variables to convert each to a 32-bit int value and then do the
    arithmetic. 

We have three primary requirements in implementing a good hash function for a
given data type: 

  - It should be deterministic—equal keys must produce the same hash value. 

  - It should be efficient to compute. 

  - It should uniformly distribute the keys. 

##############

https://www.thecrazyprogrammer.com/2017/06/hashing.html

Program for Hashing in C
Below is the implementation of hashing or hash table in C.

#include<stdio.h>

#include<limits.h>
 
/*
This is code for linear probing in open addressing. If you want to do quadratic probing and double hashing which are also
open addressing methods in this code when I used hash function that (pos+1)%hFn in that place just replace with another function.
*/
 
void insert(int ary[],int hFn, int size){

    int element,pos,n=0;
 
    printf("Enter key element to insert\n");
    
    scanf("%d",&element);
 
    pos = element%hFn;
 
    while(ary[pos]!= INT_MIN) {  // INT_MIN and INT_MAX indicates that cell is empty. So if cell is empty loop will break and goto bottom of the loop to insert element
    
        if(ary[pos]== INT_MAX)
        
            break;
            
        pos = (pos+1)%hFn;
        
        n++;
        
        if(n==size)
        
        break;      // If table is full we should break, if not check this, loop will go to infinite loop.
        
    }
 
    if(n==size)
    
        printf("Hash table was full of elements\nNo Place to insert this element\n\n");
        
    else
    
        ary[pos] = element;    //Inserting element
        
}
 
void delete(int ary[],int hFn,int size){
    /*
    very careful observation required while deleting. To understand code of this delete function see the note at end of the program
    */
    
    int element,n=0,pos;
 
    printf("Enter element to delete\n");
    
    scanf("%d",&element);
 
    pos = element%hFn;
 
    while(n++ != size){
    
        if(ary[pos]==INT_MIN){
        
            printf("Element not found in hash table\n");
            
            break;
            
        }
        
        else if(ary[pos]==element){
        
            ary[pos]=INT_MAX;
            
            printf("Element deleted\n\n");
            
            break;
            
        }
        
        else{
        
            pos = (pos+1) % hFn;
            
        }
        
    }
    
    if(--n==size)
    
        printf("Element not found in hash table\n");
        
}
 
void search(int ary[],int hFn,int size){

    int element,pos,n=0;
 
    printf("Enter element you want to search\n");
    
    scanf("%d",&element);
 
    pos = element%hFn;
 
    while(n++ != size){
    
        if(ary[pos]==element){
        
            printf("Element found at index %d\n",pos);
            
            break;
            
        }
        
        else
        
            if(ary[pos]==INT_MAX ||ary[pos]!=INT_MIN)
            
                pos = (pos+1) %hFn;
                
    }
    
    if(--n==size) printf("Element not found in hash table\n");
    
}
 
void display(int ary[],int size){

    int i;
 
    printf("Index\tValue\n");
 
    for(i=0;i<size;i++)
    
        printf("%d\t%d\n",i,ary[i]);
        
}

int main(){

    int size,hFn,i,choice;
 
    printf("Enter size of hash table\n");
    
    scanf("%d",&size);
 
    int ary[size];
 
    printf("Enter hash function [if mod 10 enter 10]\n");
    
    scanf("%d",&hFn);
 
    for(i=0;i<size;i++)
    
        ary[i]=INT_MIN; //Assigning INT_MIN indicates that cell is empty
 
    do{
    
        printf("Enter your choice\n");
        
        printf(" 1-> Insert\n 2-> Delete\n 3-> Display\n 4-> Searching\n 0-> Exit\n");
        
        scanf("%d",&choice);
 
        switch(choice){
        
            case 1:
            
                insert(ary,hFn,size);
                
                break;
                
            case 2:
            
                delete(ary,hFn,size);
                
                break;
                
            case 3:
            
                display(ary,size);
                
                break;
                
            case 4:
            
                search(ary,hFn,size);
                
                break;
                
            default:
            
                printf("Enter correct choice\n");
                
                break;
                
        }
        
    }while(choice);
    
 
    return 0;
}

##############
Program for Hashing in C++
Below is the implementation of hashing or hash table in C++.

#include<iostream>
#include<limits.h>
 
using namespace std;
 
/*
This is code for linear probing in open addressing. If you want to do quadratic probing and double hashing which are also
open addressing methods in this code when I used hash function that (pos+1)%hFn in that place just replace with another function.
*/
 
void Insert(int ary[],int hFn, int Size){
    int element,pos,n=0;
 
    cout<<"Enter key element to insert\n";
    cin>>element;
 
    pos = element%hFn;
 
    while(ary[pos]!= INT_MIN) {  // INT_MIN and INT_MAX indicates that cell is empty. So if cell is empty loop will break and goto bottom of the loop to insert element
        if(ary[pos]== INT_MAX)
            break;
        pos = (pos+1)%hFn;
        n++;
        if(n==Size)
            break;      // If table is full we should break, if not check this, loop will go to infinite loop.
    }
 
    if(n==Size)
        cout<<"Hash table was full of elements\nNo Place to insert this element\n\n";
    else
        ary[pos] = element;    //Inserting element
}
 
void Delete(int ary[],int hFn,int Size){
    /*
    very careful observation required while deleting. To understand code of this delete function see the note at end of the program
    */
    int element,n=0,pos;
 
    cout<<"Enter element to delete\n";
    cin>>element;
 
    pos = element%hFn;
 
    while(n++ != Size){
        if(ary[pos]==INT_MIN){
            cout<<"Element not found in hash table\n";
            break;
        }
        else if(ary[pos]==element){
            ary[pos]=INT_MAX;
            cout<<"Element deleted\n\n";
            break;
        }
        else{
            pos = (pos+1) % hFn;
        }
    }
    if(--n==Size)
        cout<<"Element not found in hash table\n";
}
 
void Search(int ary[],int hFn,int Size){
    int element,pos,n=0;
 
    cout<<"Enter element you want to search\n";
    cin>>element;
 
    pos = element%hFn;
 
    while(n++ != Size){
        if(ary[pos]==element){
            cout<<"Element found at index "<<pos<<"\n";
            break;
        }
        else
            if(ary[pos]==INT_MAX ||ary[pos]!=INT_MIN)
                pos = (pos+1) %hFn;
    }
    if(--n==Size)
        cout<<"Element not found in hash table\n";
}
 
void display(int ary[],int Size){
    int i;
 
    cout<<"Index\tValue\n";
 
    for(i=0;i<Size;i++)
        cout<<i<<"\t"<<ary[i]<<"\n";
}
 
int main(){
    int Size,hFn,i,choice;
 
    cout<<"Enter size of hash table\n";
    cin>>Size;
 
    int ary[Size];
 
    cout<<"Enter hash function [if mod 10 enter 10]\n";
    cin>>hFn;
 
    for(i=0;i<Size;i++)
        ary[i]=INT_MIN; //Assigning INT_MIN indicates that cell is empty
 
    do{
        cout<<"Enter your choice\n";
        cout<<" 1-> Insert\n 2-> Delete\n 3-> Display\n 4-> Searching\n 0-> Exit\n";
        cin>>choice;
 
        switch(choice){
            case 1:
                Insert(ary,hFn,Size);
                break;
            case 2:
                Delete(ary,hFn,Size);
                break;
            case 3:
                display(ary,Size);
                break;
            case 4:
                Search(ary,hFn,Size);
                break;
            default:
                cout<<"Enter correct choice\n";
                break;
        }
    }while(choice);
 
    return 0;
}

I included some posts for reference.

https://github.com/noey2020/How-to-Talk-Algorithm-Analysis

https://github.com/noey2020/How-to-Talk-Recursion

https://github.com/noey2020/How-to-Talk-Linked-Lists

https://github.com/noey2020/How-to-Talk-Queues

https://github.com/noey2020/How-to-Talk-Stacks

https://github.com/noey2020/How-to-Talk-Lists-Stacks-and-Queues

https://github.com/noey2020/How-to-Talk-Linear-Regression

https://github.com/noey2020/How-to-Talk-Statistics-Pattern-Recognition-101

https://github.com/noey2020/How-to-Write-SPI-STM32

https://github.com/noey2020/How-to-Write-SysTick-Handler-for-STM32

https://github.com/noey2020/How-to-Write-Subroutines-in-C-Assembly-STM32

https://github.com/noey2020/How-to-Write-Multitasking-STM32

https://github.com/noey2020/How-to-Generate-Triangular-Wave-STM32-DAC

https://github.com/noey2020/How-to-Generate-Sine-Table-LUT

https://github.com/noey2020/How-to-Illustrate-Multiple-Exceptions-

https://github.com/noey2020/How-to-Blink-LED-using-Standard-Peripheral-Library

I appreciate comments. Shoot me an email at noel_s_cruz@yahoo.com!

Hire me!

Have fun and happy coding!
