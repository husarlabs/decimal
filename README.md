# decimal

## This fork

This fork changes:
* adds SetBSON and GetBSON (for mgo library)
* sets MarshalJSONWithoutQuotes=true
* adds decoder and encoder functions for official mongo driver
* adds go.mod


[![Build Status](https://travis-ci.org/shopspring/decimal.png?branch=master)](https://travis-ci.org/shopspring/decimal) [![GoDoc](https://godoc.org/github.com/shopspring/decimal?status.svg)](https://godoc.org/github.com/shopspring/decimal) [![Go Report Card](https://goreportcard.com/badge/github.com/shopspring/decimal)](https://goreportcard.com/report/github.com/shopspring/decimal)

Arbitrary-precision fixed-point decimal numbers in go.

NOTE: can "only" represent numbers with a maximum of 2^31 digits after the decimal point.

## Features

 * the zero-value is 0, and is safe to use without initialization
 * addition, subtraction, multiplication with no loss of precision
 * division with specified precision
 * database/sql serialization/deserialization
 * json and xml serialization/deserialization

## Install

Run `go get github.com/shopspring/decimal`

## Usage

```go
package main

import (
	"fmt"
	"github.com/shopspring/decimal"
)

func main() {
	price, err := decimal.NewFromString("136.02")
	if err != nil {
		panic(err)
	}

	quantity := decimal.NewFromFloat(3)

	fee, _ := decimal.NewFromString(".035")
	taxRate, _ := decimal.NewFromString(".08875")

	subtotal := price.Mul(quantity)

	preTax := subtotal.Mul(fee.Add(decimal.NewFromFloat(1)))

	total := preTax.Mul(taxRate.Add(decimal.NewFromFloat(1)))

	fmt.Println("Subtotal:", subtotal)                      // Subtotal: 408.06
	fmt.Println("Pre-tax:", preTax)                         // Pre-tax: 422.3421
	fmt.Println("Taxes:", total.Sub(preTax))                // Taxes: 37.482861375
	fmt.Println("Total:", total)                            // Total: 459.824961375
	fmt.Println("Tax rate:", total.Sub(preTax).Div(preTax)) // Tax rate: 0.08875
}
```

## Documentation

http://godoc.org/github.com/shopspring/decimal

## Production Usage

* [Spring](https://shopspring.com/), since August 14, 2014.
* If you are using this in production, please let us know!

## FAQ

#### Why don't you just use float64?

Because float64s (or any binary floating point type, actually) can't represent
numbers such as 0.1 exactly.

Consider this code: http://play.golang.org/p/TQBd4yJe6B You might expect that
it prints out `10`, but it actually prints `9.999999999999831`. Over time,
these small errors can really add up!

#### Why don't you just use big.Rat?

big.Rat is fine for representing rational numbers, but Decimal is better for
representing money. Why? Here's a (contrived) example:

Let's say you use big.Rat, and you have two numbers, x and y, both
representing 1/3, and you have `z = 1 - x - y = 1/3`. If you print each one
out, the string output has to stop somewhere (let's say it stops at 3 decimal
digits, for simplicity), so you'll get 0.333, 0.333, and 0.333. But where did
the other 0.001 go?

Here's the above example as code: http://play.golang.org/p/lCZZs0w9KE

With Decimal, the strings being printed out represent the number exactly. So,
if you have `x = y = 1/3` (with precision 3), they will actually be equal to
0.333, and when you do `z = 1 - x - y`, `z` will be equal to .334. No money is
unaccounted for!

You still have to be careful. If you want to split a number `N` 3 ways, you
can't just send `N/3` to three different people. You have to pick one to send
`N - (2/3*N)` to. That person will receive the fraction of a penny remainder.

But, it is much easier to be careful with Decimal than with big.Rat.

#### Why isn't the API similar to big.Int's?

big.Int's API is built to reduce the number of memory allocations for maximal
performance. This makes sense for its use-case, but the trade-off is that the
API is awkward and easy to misuse.

For example, to add two big.Ints, you do: `z := new(big.Int).Add(x, y)`. A
developer unfamiliar with this API might try to do `z := a.Add(a, b)`. This
modifies `a` and sets `z` as an alias for `a`, which they might not expect. It
also modifies any other aliases to `a`.

Here's an example of the subtle bugs you can introduce with big.Int's API:
https://play.golang.org/p/x2R_78pa8r

In contrast, it's difficult to make such mistakes with decimal. Decimals
behave like other go numbers types: even though `a = b` will not deep copy
`b` into `a`, it is impossible to modify a Decimal, since all Decimal methods
return new Decimals and do not modify the originals. The downside is that
this causes extra allocations, so Decimal is less performant.  My assumption
is that if you're using Decimals, you probably care more about correctness
than performance.

## License

The MIT License (MIT)

This is a heavily modified fork of [fpd.Decimal](https://github.com/oguzbilgic/fpd), which was also released under the MIT License.
