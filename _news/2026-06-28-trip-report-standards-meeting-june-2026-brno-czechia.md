---
title: "Trip Report Standards Meeting June 2026 Brno, Czechia "
date: 29/06/2026T02:51
author: Vasil Pashov
---
The June 2026 ISO C++ standards meeting in Brno, Czech Republic, held between 08.06 and 13.06.2026 is now over. It was hosted by Mendel University and was attended by approximately 200 people, almost half of whom were online.
## Directions for C++29
The DIS (Draft International Standard) for C++26 was officially forwarded to ISO earlier in June, meaning that this is the first meeting where C++29 is the main focus. The Direction working group stated the goals for C++29.

C++26 ships large features such as reflections and contracts which present significant change to the language. C++26 made only the first steps towards those features, providing a minimal viable product. C++29 will  be a smaller release focusing on filling the gaps, completing papers that didn't make it to 26, thus giving the implementors time to catch up. C++29 will also focus on improving safety and security of C++.
## What was formally accepted in Brno
### CWG
20 motions were accepted and forwarded to the new working draft.

####  [P3596](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3596r2.pdf) (Undefined Behavior and IFNDR Annexes) to the C++ Working Paper.
This is a great source for learning. The authors of the paper have analyzed and categorized all mentions of UB and IFNDR in the standard. The new Annex D will contain an example for each case, so anyone finding the wording in the standard hard to swallow can refer to the annex for a concrete example. Going forward authors of new papers will be advised to extend the annex should they decide to add new UB or new IFNDR.

#### [P2287](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2287r5.html) (Designated-initializers for Base Classes) to the C++ Working Paper.
Extends designated initialization for derived classes, making this possible

```cpp
struct A {int a;};
struct B : A {int b};

B my_b{.a = 1, .b = 2} // Note that this allowes using the name "a" of the member of the base class
```

####  [P3424](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3424r1.pdf) (Deallocation Functions with Throwing Exception Specification Are Ill-formed) to the C++ Working Paper.
Placing `noexcept(false)` on deallocation functions (operator delete) is now hard error. Prior to this, it was UB.

#### [P3822](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3822r1.html) (Conditional noexcept specifiers in compound requirements) to the C++ Working Paper.
Makes it easier to require a noexcept specifier in a requires-expression.

```cpp
template<typename F, bool noexc>
concept invocable = requires(F f) {
  { f() } noexcept(noexc);
};
```

#### [P3097](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3097r2.pdf) (Contracts for C++: Virtual functions) to the C++ Working Paper.
C++26 shipped only with an MVP for contracts which does not support adding contracts on a virtual function. The initial design was controversial and could not gain consensus, thus the committee chose to give the authors some time to refine the proposal.
By design contracts are **not** inherited. When there is a virtual function we have 2 sets of contracts. The contract of the interface (the contract of the statically typed function in code) and the contract of the virtually dispatched functions. Both are evaluated in the following order:
1. Evaluate preconditions of the base 
2. Evaluate the preconditions of the virtually dispatched function
3. Evaluate the postconditions of the virtually dispatched function
4. Evaluate the post conditions of the base
It can be illustrated with an example

```cpp
struct A {
	virtual void f(int param) pre(param > 10);
};

struct B : A {
	void f(int param) override pre(param < 100);
};

struct C : B {
	void f(int param) override pre(param < 1000);
}

void f1(A& a) {
	a.f(9); // test 1
	a.f(50); // test 2
	a.f(500)// test 3
}

void f2(B& b) {
	b.f(9); // test 1
	b.f(50); // test 2
	b.f(500)// test 3
}

void f3(C& c) {
	c.f(9); // test 1
	c.f(50); // test 2
	c.f(500)// test 3
}

int main() {
	A a;
	B b;
	C c;
	// As there is no virtual dispatch only one precondition will be evaluated param > 10
	// test 1 - fails (param = 9 < 10)
	// test 2 - succeeds param = 50 > 10
	// test 3 - succeeds param = 500 > 10
	f1(a);
	// Virtual dispatch is going to happen. First will evaluated param > 10 and then param < 100
	// test 1 - fails (param = 0 < 10)
	// test 2 - succeeds param = 50 > 10 && param < 100
	// test 3 - fails param = 500 > 10 (ok) param = 500 > 100 (fails)
	f1(b);
	// Again virtual dispatch is going to happen. Will check against A's and C's contracts
	// test 1 - fails param = 9 < 10
	// test 2 - succeeds param = 50 > 10 && param = 50 < 1000
	// test 3 - succeeds param = 500 > 10 && param = 500 < 1000
	f1(c);
	
	// No virtual dispatch will evaluate only B's preconditions param < 100
	// test 1 - succeeds param = 9 < 100 (does not check param > 10)
	// test 2 - succeeds param = 50 < 100 (does not check param > 10)
	// test 3 - fails param = 500 > 1000 (fails) (does not check param > 10) 
	f2(b);
	// Virtual dispatch will happen. It will check both B's and C's contrats. A's contracts will not be checked
	// test 1 - succeeds param = 9 < 100 && param = 9 < 1000 (does not check param > 10)
	// test 2 - succeeds param = 50 < 100 && param = 50 < 1000 (does not check param > 10)
	// test 3 - fails param = 500 > 100
	f2(c);
	
	// No virtual dispatching. Only C's contracts are checked all tests are passing.
	f3(c);
}
```

This sparked quite the discussion. Some members did not agree to the proposal that the base class contracts should be checked. Two arguments emerged
1. It's counter-intuitive/hard to teach, as virtual functions do not call any parent methods by default. So it's not obvious that a contract will call part of the parent's code
2. It might be restrictive. When writing a function expecting a base class one cannot possibly know the input range of all derived classes. As in the example above `C::f` extends the input range of `B::f`.
3. No other language has implemented contracts the way `C++` is implementing them
The counterargument presented is that contracts should not be concerned if inheritance will widen or narrow the input range. Instead contracts should enforce what you see in code, if a function expects something of type `T` it should at the very least adhere to the contract of `T`. It is true that `C++` implemented contracts differently compared to `Ada`, `D`, `Eiffel` however the languages are implemented differently. The paper explores in details the differences and the reasoning behind the proposal.

####  [P3670](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3670r1.pdf) (Pack Indexing for Template Names) to the C++ Working Paper.
C++26 allows indexing of pack parameters. However it does not extend to indexing parameter pack of template parameters. This paper allows that

```cpp
// Allowed in C++26
template<typename... T>
struct S26 {
	using First = T...[0];
};
S26<int, float> s26;
static_assert(std::same_as<decltype(s26)::First, int>);

// Not allowed in C++26, proposed by the paper
template<template<typename>... TT>
struct S29 {
	using First = TT...[0]<int>;
};
S29<std::vector, std::unique_ptr> s29;
static_assert(std::same_as<decltype(s29)::First, std::vector<int>>);
```

#### [P3540](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3540r2.html) (#embed offset parameter) to the C++ Working Paper.
Allow skipping bytes before embedding them. The option is already supported as an extension in both clang and GCC

#### [P3668R3 ](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3668r3.html)  Defaulting Postfix Increment and Decrement Operations
Allows a compiler-generated postfix increment/decrement based on a defined prefix operator. 

```cpp
class foo{
    int member;
public:
    constexpr foo& operator++(){
        ++member;
        return *this;
    }
    constexpr foo operator++(int) = default;
};
```

#### Defect reports
Defect reports are related to the wording of the standard itself. Sometimes people find that the wording formally allows cases that should not be allowed, or the wording is ambiguous or it does not mandate the correct thing. They are generally not intended to change behaviour but they definitely can change both runtime and static behaviour.  Note that DRs can be targeted at older versions (e.g. some were targeted at C++20). Here is a list of all DRs that were accepted
1. P4271 Core Language Working Group "ready" Issues for the June, 2026 meeting
2. [P3899 Clarify the behavior of floating-point overflow](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3899r2.html) 
3. [P2434 Nondeterministic pointer provenance](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2434r4.html) 
4. [P3347 Invalid Pointer Operations](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3347r5.pdf) 
5. [P3658 Adjust identifier following new Unicode recommendations](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3658r1.pdf) 
6. [P3950 return_value & return_void Are Not Mutually Exclusive](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3950r0.pdf) 
7. [P3733 More named universal character escapes](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3733r1.html) 
8. [P3847 Lexical order for lambdas](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3847r0.html) 
9. [P2243 Language linkage for templates](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2243r0.html) 
10. [P4101 Consteval-only Values for C++26](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p4101r0.html) 
11. [P2414 Pointer lifetime-end zap proposed solutions](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2414r10.pdf) 

### LWG

[P3091R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3091r5.html) (Better Lookups for `map`, `unordered_map`, and `flat_map`) to the C++ working paper.
Provides Python-like lookup methods for maps in STL.  `operator[]` is by far the most convenient way to get a value out of a map but it creates an entry in the map which although a known behavior was a source of bugs. It also cannot be used on const maps. Lookup returns `std::optional<ValueType&>` and can be used like so:

```cpp
std::unordered_map<std::string, int> my_map;
my_map.lookup("key").value_or(0);
```

[P3125R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3125r5.html) (constexpr pointer tagging) to the C++ working paper.
Not all bits of a pointer are actually used in addressing. This gives us the ability to use the lower bits to attach some additional information to a pointer.

P4258R0 (C++ Standard Library Ready Issues to be moved in Brno, Jun. 2026) to the C++ working paper.

[P3319R6](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3319R6.pdf "2026-06 Brno-StrawPolls - P3319R6.pdf") (Add an `iota` object for `simd` (and more)) to the C++ working paper.

[P3798R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3798r1.html) (The unexpected in `std::expected`) to the C++ working paper.

[P3052R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3052r2.html) (`view_interface::at()`) to the C++ working paper.

[P4206R0](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p4206r0.html) (Revert string support in `std::constant_wrapper`) to the C++ working paper.

[P3395R6](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3395-error_code_formatting.html "2026-06 Brno-StrawPolls - P3395-error code formatting.html") (Fix encoding issues and add a formatter for `std::error_code`) to the C++ working paper.

[P3505R4](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3505.html "2026-06 Brno-StrawPolls - P3505.html") (Fix the default floating-point representation in `std::format`) to the C++ working paper.

[P3154R3](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3154R3.html "2026-06 Brno-StrawPolls - P3154R3.html") (Deprecating signed character types in iostreams) to the C++ working paper.

[P3428R4](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3428R4_Hazard_Pointer_Batches.pdf "2026-06 Brno-StrawPolls - P3428R4 Hazard Pointer Batches.pdf") (Hazard Pointer Batches) to the C++ working paper.

[P3248R5](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3248R5-intptr.html "2026-06 Brno-StrawPolls - P3248R5-intptr.html") (Require `[u]intptr_t`) to the C++ working paper.

[P3793R2](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3793R2.html "2026-06 Brno-StrawPolls - P3793R2.html") (Better shifting) to the C++ working paper.

[P3242R4](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3242R4.html "2026-06 Brno-StrawPolls - P3242R4.html") (Copy and fill for `mdspan`) to the C++ working paper.

[P3692R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3692r4.pdf) (How to Avoid OOTA Without Really Trying) to the C++ working paper.

[P3104R6](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3104R6.html "2026-06 Brno-StrawPolls - P3104R6.html") (Bit permutations) to the C++ working paper.

[P3772R2](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3772R2.html "2026-06 Brno-StrawPolls - P3772R2.html") (`std::simd` overloads for bit permutations) to the C++ working paper.

[P2019R9](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P2019R9.pdf "2026-06 Brno-StrawPolls - P2019R9.pdf") (Thread attributes) to the C++ working paper.

[P3785R1](https://wiki.isocpp.org/uploads/2026-06_Brno-StrawPolls_-_P3785R1.html "2026-06 Brno-StrawPolls - P3785R1.html") (Library Wording Changes for Defaulted Postfix Increment and Decrement Operations) to the C++ working paper.

### Some other papers the committee discussed and will keep working on in future meetings
This is a non-exhaustive list of papers I found interesting. They were discussed but were not included in the standard this time. The committee will continue working on them in the future. Papers are not listed in any particular order.

#### [P4222 An initialization profile](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p4222r0.pdf)
Profiles are a big topic meant to enforce stricter rules so that the code can achieve a specific guarantee. The idea is similar to how a set of rules can be enforced via static analyzers.

Bjarne Stroustrup presented what would be the rules that an initialization profile must enforce. The profile mandates that every object must either be initialized or explicitly marked by the compiler as not being initialized. Uninitialized objects must not be read.

There are several papers specifying the rules for different profiles. However, it is not yet clear how profiles are going to be standardized, how they are going to be enforced or the wording needed to be placed in standard.

#### [P3568 `break label;` and `continue label;`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3568r2.html)
Allow placing goto-like labels on loops (similar to how Java allows it) to enable easy breaking out of nested loops. This is already being discussed by the `C` language committee and there are plans to add it in `C`. This paper actually mimics the paper for `C`. The only concern here is that we add it before `C` and then `C` decides the change something in the last minute we'll end up with incompatibility in the languages (in the worst case) or we'll have to add another paper to fix the draft before C++29.

#### [P3385 Attributes reflection](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3385r7.html)
The feature was found generally useful. Not all attributes can be reflected. Some implementors raised concerns regarding implementability of the feature.

#### [P4033R0  Synthesizing enum at compile time with define_enum](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p4033r0.html)
Allows generating enums similar to how we can generate aggregates via `define_aggregate`

#### [P3100 A framework for systematically addressing undefined behaviour in the C++ Standard](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3100r6.pdf)
Improving the safety of C++ will be important in future. This paper examined all undefined behaviors (79 instances) in the language and proposes a way to prevent them. Most of the UBs cannot be checked at compile time, the suggestion for lots of the cases is to enforce the compiler to insert a contract check at places where UBs can happen. The committee decided to discuss the paper "line-by-line" in a series of telecons.

#### [P3850 A proposed plan for extending Contracts in C++29](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2026/p3850r0.pdf)
The contracts in C++26 are only a minimal viable product. This paper represents the plan of the two authors to extend the contracts. Note: other papers by other authors will be accepted for a review. This paper was only meant to present the plan of the two authors for the future.

#### [P1974R1 Persistent constexpr allocation]([Persistent constexpr allocation](https://isocpp.org/files/papers/P1974R1.pdf))
This will allow memory allocated at compile time to be used at runtime, making this possible.

```cpp
constexpr auto f() {
	std::vector<int> a;
	a.push_back(1);
	a.push_back(2);
	return a;

}

constexpr void g() {
	// Not possible in C++26. Deallocation must happen by the end of the constant expression f();
	constexpr auto v = f();
}

int main() {
	g();
}
```
