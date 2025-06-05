+++
date = '2025-05-03T21:40:39+05:30'
name = 'Utkarsh Sharma'
title = "Membership proofs for a continous set of elements"
draft= true
+++

# Membership proofs for a continous set of elements

## Why?

I encountered this problem when I was making a privacy preserving [whitsleblowing](https://www.dni.gov/ICIG-Whistleblower/what-is.html#:~:text=WHAT%20IS%20WHISTLEBLOWING?,within%20IC%20programs%20and%20activities.) platform for a hackathon. The main motive of the platform was to have whitsleblower prove his knowledge without revealing his identity. Well with a caveat that he must have an email backing his claim.

Now given the contraints of email all that is to be done was to prove in zero knowledge that the whitsleblower got an email from someone with email address `@_` and had some content that the he claims to know. For example you can imagine a scenario where you were fired wrongfully via an-email which contains `you are fired because I hate you`.

The first problem was solved through [zk-email](https://prove.email/) which verifies that it is a valid email through [**DKIM Signature**](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) verification in circom and extracts the *from* email address from the email header.

For the second problem I need to prove in zero knowledge that the email body contained the claimed text which if we convert body and claimed content into an array of ascii elements. This boils down to proving in zero knowledge that a larger array contains a smaller array

## A problem so trivial at surface

Let's say we have an arr $A$ then a continous set of elements are
$$S = {A[i], A[i+1] ... A[i+c]}$$ where $c$ is the lenght of the array and $i$ is the starting index and in zero-knowledge (or in a circuit) we want to prove that $S$ is a subset of $A$.

So in pseudocode we just have to do:

```rust {class="my-class" id="my-codeblock" lineNos=inline tabWidth=4}
    for ( index := 0..c) {
        assert S[i] == A[i + m] 
    }
```

This seems like a very trivial problem but if we were to write this as circom circuit it would not work.

## Constraints of writing circuits in circom

You can think of them as a way of defining a computation in a zk-friendly way. Any computation that can be converted into a circuit can be proved in zero knowledge.

{{< light >}}
If you are familiar with Zk cryptography or have tried to dabble into them you must have heard of the term *circuit*. Well it's not circuit *per say* as there is no current or voltages involved but here circuit is actually a short name for an arthematic or boolean circuit. Which are logical constructs made by mathematicians to define a formal language for any problem. The rareskills ZK book is an exellent resource for understanding this.
{{< /light >}}

The main problem is that it is not dynmaic at all. From the **circom docs**:
> when constraints are generated in any block inside an if-then-else or loop statement, the condition cannot be unknown
An Unkown is bascially a value that is input to the circuit or something that is dynamic. Thus if we convert our pseudocode directly into circom with something like this:

```circom
template thisDoesNotWork(n,k){
    signal input big_arr[n];
    signal input small_arr[k];
    signal input starting_index;

    for(var i=0; i<k; i++){
        big_arr[starting_index + i] === small_arr[k];
    }
}
```

If we try to compile this code we get the following error:
![this is the image](/img/cont_set/1.png)
So as expected this does not compile as the constraint becomes non quadratic due to an unknown at compile time.

## Writing a trivial working circuit

Let's write a circuit with the trivial logic. So one thing you need to keep in mind is when writing circuit if your program has branching you have to convert that piece of code into arthematic constraints. for example:
