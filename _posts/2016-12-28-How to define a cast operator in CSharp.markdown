---
layout: post
title:  "How to define a cast operator in C#"
date:   2016-12-28 20:19:00 -0600
categories: programming
---
In this article I’m going to explain what are cast operators, how to define them, some pitfalls, and if we really need them.

**What are cast operators?**

Cast operators, or conversion operators (which is a more appropriate name in C#) are ways that the language provides to convert one data type to another.
These conversions happen in statements: by doing arithmetic, an assignment, or by passing the value to a function. And can occur either implicitly or explicitly.
One example of an implicit conversion is when we try to assign an `int` value to a `double` one:

``` cs
int foo = 10;
double bar = foo;
```

Here, `foo` is converted implicitly to a double which makes possible the assignment. This happens automatically since it’s a widening conversion; meaning that the value being assigned occupies less memory (bytes) than the value to which is assigned. So it’s perfectly safe to make this assignment, since there’s no risk of losing data or precision (integer types takes up 32 bits in memory, and double are 64 bits).
But what if we need to convert a bigger type to a smaller one? For example, a `double` to `int`:

``` cs
double foo = 11.3;
int bar = foo;
```

That would cause a compiler error, showing an error with the form of “Can’t convert implicitly the type ‘double’ to ‘int’”. The reason is because if the compiler had made that conversion it will result in a loss of data (narrowing conversion), because `doubles’` range are bigger than `int`, and integers can’t represent decimal point values.

So what we need is a way to tell the compiler that we know what we’re doing and let us convert the type. This is where conversion operators comes in handy.

Conversion operators has the form of: **(typeToConvert) value**.

So if we need to make last example legal, we need to use the explicit conversion:

``` cs
double foo = 11.3;
int bar = (int)foo;
```

Here, `foo` is casted (converted) to an `int` type and then assigned to `bar` variable.

So, to resume a bit: Implicit conversions happens when there’s no risk of losing data, nor throwing exceptions during the conversion. And explicit ones are those we specify in a form of a cast and can result on losing precision.


**How to overload a conversion operator.**

Let’s say we have a Month class, representing all the months from “January” through “December.” Something like this:

``` cs
class Month
    {
        string[] _items = {
            "January", "February", "March",
            "April", "May", "June",
            "July", "August", "September",
            "October", "November", "December"
        };
    }
```

>Note that here we could use an static readonly array of strings, or even better; an enumeration instead of this. Our example might not have the best design choices, but the purpose here is demonstrate how to overload the conversions so bear with me.

Now, what if we want to store the month in a DB or a file?, mapping the months to an `int` value might be a good idea since we could save space and improve performance (or maybe to store it as a Foreign Key). So the month “January” is going to be the number 1, “February” is 2, and so on.

One way to achieve this is through overloading conversion operators.

Conversion operators have the following syntax:

``` cs
accessModifier static explicitOrimplicit operator returnType(convertFrom)
{
    // …
}
```

And have the following properties:
-	Conversions with the `implicit` keyword occur automatically when it is required.
-	Conversions with the `explicit` keyword needs a cast to make the conversion.
-	They all need the static modifier.
-	Either (but not both) the returned value or the converted from value must be the containing type.

So to declare a conversion from `Month` type to an `int`, we can do the following:

``` cs
public static implicit operator int(Month m)
{
	for (int i = 0; i < m._items.Length; ++i)
	{
		if (m._items[i] == m.Current)
		{
			return i + 1;
		}
	}

	return 0;
}
```
The important part here is the signature of the overlading. The cast is marked as implicit, so making an assignment from `Month` type to `int` is possible without a cast:

``` cs
var month = new Month();
month.Current = "February";
int bar = month;

Console.WriteLine("Month int is: {0}", bar);
```

Will print: “Month int is: 2”.

In my opinion, here it’s better to use implicit rather than explicit because all the `Month` values can be converted to int, even if the `Month` hasn’t been initialized (therefore containing an invalid `Current` value.) In that case we return a zero if the `Month` is invalid.

Now, how do we convert an `int` to a `Month`?
Since all Months can be represented in integers, but not all integer’s values can represent a `Month`, it’s better to use an explicit conversion.
We define it as the following:

``` cs
public static explicit operator Month(int m)
{
	var month = new Month();
	if (m > 0 && m <= month._items.Length)
	{
		month._current = month._items[m - 1];
		return month;
	}

	return null;
}
```

Note that if we provide an unvalid int --that is, outside the range of Month values-- (< 1 or > 12) we return a null instead a `Month`.

Now we cast the `int` specified to a `month`.

``` cs
int m = 7;
var month2 = (Month)m;

if (month2 != null) {
	Console.WriteLine("int to Month is: {0}", month2.Current);
} else {
	Console.WriteLine("Fail to convert int to Month type");
}
```
That will print: "int to Month: July"

You can check the full code [here](https://gist.github.com/yureru/3b9e298aab2230d78de474d4d17f4074)

**Do we really need to overload conversion operators?**

It’s a matter of choice, but I rather declare functions to convert those values. For example if we need to convert the `Month` value to an integer I would define a `Month.ToInt()` function. For achieving the reverse thing I’ll probably declare a constructor or property (with validations) that accepts an `int` value.
