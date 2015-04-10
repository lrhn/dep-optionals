#Positional and named optional parameters.

## Contact information

Name: Lasse R.H. Nielsen
E-mail: lrn@google.com
[DEP Proposal Location][]

## Summary

Allow both optional positional and named parameters on the same function.
Example:
```dart
   int parse(String int, [int start, int end],
             {int radix, void onError(String s)});
```

## Background and motivation
The [Dart standard][] currently allows either optional positional parameters or optional named parameters in a function signature, but not both.

This causes API designers to have to choose either one or the other in cases where a combination might be the best solution, or to omit adding optional parameters that would otherwise make sense because they are not compatible with existing parameters. See examples at the end of this document.

APIs lock the designer in more than necessary because a choice of adding an optional parameter also prevents all further additions of the other kind of optional parameters, something that would otherwise be a non-breaking change.

Function types with both kinds of optional parameters means that you can always implement the intersection of two interfaces, even if they define the same name with different signatures.
Example:
```dart
abstract class C { int foo(int x, [int y]); }
abstract class D { int foo(int x, {int z}); }
abstract class E implements C, D {}
class F implements E {
  int foo(int x, [int y], {int x}) { … }
}
```

Currently it is not possible to create a function signature that satisfies both interfaces - even if the interface of E actually contains a function with that signature!

Basically, the restriction is an annoying non-orthogonal design. If adding an extra optional named or positional parameter is always possible, then the function design space would be fully orthogonal, instead of only supporting three of the four quadrants of {no positional, positional} x {no named, named} matrix.

## Proposal
All function declarations should allow both optional positional and optional named parameters.

The syntax for a parameter list changes by allowing named parameters after optional positional parameters. Change the production:
```
optionalFormalParameters:
      optionalPositionalFormalParameters
    | namedFormalParameters ;
```
to:
```
optionalFormalParameters:
      optionalPositionalFormalParameters  ( ‘, ’ namedFormalParameters)?
    | namedFormalParameters ;
```

This is the only change to the language syntax. Call points need no change since they are unaware of whether the function they call has optional positional parameters or not - the all merely pass a number of positional and a choice of named arguments and let the function figure out whether the arguments match the parameters.

Section "Binding Actuals to Formals" needs to be rewritten, since it assumes that there can't be both positional and named optional parameters. The rewrite will require splitting the formals into three groups instead of two (and then using that the optional parameters are all either named or positional), but the behavior is the same, and I expect the rewritten version to be less complex (except for requiring an extra subscript variable). Similar for other places in the specification like the least-upper-bound of function types or similar places that are handled by cases.

Sections dealing with extraction of functions and constrcutors can also be reduced to one case instead of two.

## Implications
This requires rewriting all implementations of the language to support the extra functionality. How big a change that is depends on how deeply the assumption that functions only have one kind of optional parameters is embedded in the design.

Existing libraries will now look badly designed if they are using only one kind of optional parameters where using both would be better. It would be great to be able to do a clean-up of existing libraries, which will be a breaking change and have to be delayed until Dart 2.0. That does not mean that this change can't be added earlier, only that the full effect of it won't be felt until we get a chance to remove the existing consequences of the restriction. At least we won't be adding more.

## Examples
### DateTime constructor (dart:core)
```dart
   DateTime([int year, int month, int day, int hours,
             int minutes, int seconds, int milliseconds,
             bool isUtc]);
```
Here the isUtc parameter would have been much better as a named parameter. We generally use named parameters for boolean flags, but here it wasn't possible because of the other positional optional parameters.
###int/double.parse (dart:core)
```dart
int parse(String source, {int radix, int onError(...));
double parse(String source, [double onError(...)]);
```
The signatures already differ in that onError is a positional parameter in one function and a named parameter in the other. That was just bad design. If possible, the onError would likely have been positional in both cases, but should really have been named. Looking forward, it would be useful to allow parsing a sub-string of a larger string, changing the signatures to:
```dart
int parse(String source, [int start = 0, int end],
                         {int radix, int onError(...));
double parse(String source, [int start = 0, int end],
                            {double onError(...)});
```
### Stream.listen (dart:async)
```dart
   ... listen(void onData(T data),
              {void onError(...),
               void onDone,
               bool cancelOnError: false});
```
Here the onData parameter isn't always necessary (it very often is, but exceptions happen). If possible, it would have been an optional positional parameter, but since it isn't possible, you have to explicitly pass `null` in those cases.

## Deliverables

### Language specification changes

The [changed specification] is included with this proposal.

### A working implementation

All Dart parsers and compilers must be updated to accept the changed syntax.

Runtime implementations must be able to call functions with both kinds of optional parameters.

## Patents rights

TC52, the Ecma technical committee working on evolving the open [Dart standard][], operates under a royalty-free patent policy, [RFPP][] (PDF). This means if the proposal graduates to being sent to TC52, you will have to sign the Ecma TC52 [external contributer form]() and submit it to Ecma.

[DEP Proposal Location]: https://github.com/lrhn/dep-optionals/
[changed specification]: https://github.com/lrhn/dep-optionals/dartLangSpec.tex
[dart standard]: http://www.ecma-international.org/publications/standards/Ecma-408.htm
[rfpp]: http://www.ecma-international.org/memento/TC52%20policy/Ecma%20Experimental%20TC52%20Royalty-Free%20Patent%20Policy.pdf
[form]: http://www.ecma-international.org/memento/TC52%20policy/Contribution%20form%20to%20TC52%20Royalty%20Free%20Task%20Group%20as%20a%20non-member.pdf
