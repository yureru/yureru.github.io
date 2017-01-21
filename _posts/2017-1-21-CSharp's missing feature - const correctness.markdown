---
layout: post
title:  "C#'s missing feature: const correctness"
date:   2017-1-20 11:29:00 -0600
categories: programming
---
Ever since I started using C# I’ve noticed how well this language was (and keeps being) designed. Almost every feature, from unsigned data types up to asynchronous methods, has been a delight to work with. But coming from languages like C and C++, there’s a feature I really miss, and that is const correctness.

Const correctness is just a way to specify a parameter that shouldn’t be modified in that given function, therefore documenting your program in such a way that makes clear you are just reading from that variable, minimizing side effects, and making it less prone to bugs.

For example, in C we might have a struct Person with two members: Name and Age. And a function print() that prints the contents of that struct.

``` c
#include <stdio.h>

struct Person
{
	char* Name;
	int Age;
};

void print(const struct Person *item);

int main(void)
{
	struct Person item;
	item.Name = "Anna";
	item.Age = 23;

	print(&item);

	return 0;
}

void print(const struct Person *item)
{
	printf("%s %d", item->Name, item->Age);
}
```


So making the print’s parameter to const is a great example of the usage of const correctness, since all we want to do is read that object. If we make an assignment to the struct’s members, like in the following example, we would get a compile error stating that we can’t change the members due the const modifier.

``` c
void print(const struct Person *item)
{
	item->Name = "Lucy"; // error at compile time
	printf("%s %d", item->Name, item->Age);
}
```

You might have several questions regarding about the utility of const, for example:

-	*Why would I need to state that I won’t change that variable in the function?, Why not just make sure we don’t modify it?*

Well, in a small function like the print one above, it might be really easy to detect any assignment, now imagine that you have a long function with several parameters, things start to get tougher. The good thing about using const is that clearly states you shouldn’t make modifications to that object, and as I’ve already said: **It documents your program.** And self documenting code is good.

-	*Why not just pass it by value?, instead of passing it by reference (or pointer). That way we make sure the changes doesn’t affects the original value, don’t we?*

Kinda, but if it’s a large object it might cause overhead copying all the members/properties of the object. And it doesn’t solves the problem that we might accidentally end up changing that parameter in the function.


What made me miss the feature was a mistake I did while learning WCF, basically I was translating a Data Transfer Object to a Business Domain Object, but I end up using the product parameter in the assignment instead of the productBDO (due my avid use of Intellisense auto-complete).

This was the code:

![Mistake_cost_correctness]({{ site.url }}/assets/post_images/mistake_const_correctness.png)

So after running the code, and seeing results that I wasn’t expecting I had to debug the application and wasting some minutes perhaps, which could be saved if we had the const option.

You can now look up in the net about what where the decisions behind not allowing this functionality and ways to work around this, but to be honest, it’s more work than anything you will gain from it.

