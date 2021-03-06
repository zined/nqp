nqp/examples/rubyish
====================

Ruby subset extended from the `rubyish-4` example, as introduced in the Edument
[Rakudo and NQP internals course](https://github.com/edumentab/rakudo-and-nqp-internals-course).

Example usage:
```
    % nqp-p rubyish.nqp -e'puts "Hello World!"'
    % nqp-p rubyish.nqp examples-rubyish/template.rbi
    % nqp-p rubyish.nqp  # repl mode
    > puts 37 + 5
```
Implemented:
- simple strings 'Hello World!' %q{...}
- interpolating strings: "number #{37+5}" %Q{Hello #{planet}!}
- quoted words: `%w[aa bb cc]` 
- basic scoping, including $globals, @class_instance and @@package variables
- conditional blocks: `if ... then ... elsif ... else ... endif`, `unless..end`
- nqp opcode calls: `nqp::sleep(5)`
- a few built-ins: `abort`, `print`, `puts`, `sleep`
- a couple of methods: `.call` and `.nil?`
- infixish assignments: `+=` `-=` `*=` ...
- simple classes and objects with attributes.
- inheritance (no mixins yet) - see [inheritance.t](t/inheritance.t)
- `while` and `until` loops
- statement modifiers `if` `unless`, `while`, `until` e.g.: `puts 42 if true`
- basic arrays and hashes
- for loops on arrays: `for val in [10, 20, 30] do puts val end`
- for loops on hash keys: `h = {'a'=>10, 'b'=>20}; for k in h do puts h{k} end`
- lambda blocks/closures: `def make_counter(n,incr) ; n-=incr; lambda { n += incr }; end`
- lightweight eRuby like templating, see [template.rbi](examples-rubyish/template.rbi)
- heredocs, literal `<<EOF ... EOF` and interpolating `<<"END" ... END`
- code block arguments `even = grep(arr) {|n| n % 2 == 0}` -- see [functional.t](t/functional.t)
- package constants `Trig::PI = 3.1415926`

Notes:
------

Handy Options:

    % nqp-p rubyish.nqp --target=parse -e'puts "Hello World!"'  # dump parse
    % nqp-p rubyish.nqp --target=ast   -e'puts "Hello World!"'  # dump ast
    % nqp-p rubyish.nqp --target=ast    # REPL mode, dump ASTS
    > puts 42

To run tests:
```
    % prove -v -e'nqp-p rubyish.nqp' t
```

Strings and truth values are Perlish rather than Rubyish:
- `+` always does addition (doesn't concatenate strings)
- `~` has been introduced as the concatenation operator
- `>`, `==`, `<=` ... only do arithmetic comparisons
- `gt`, `eq`, `le` ... do string comparisons
- 0, '0', '' are false in a boolean context.
- hash dereferencing is via angle braces: `puts fruit<apples>` or
curlies `puts fruit{'bananas'}`

nqp op-codes can be called like regular functions. E.g.
```
    puts nqp::if(2+2 == 4, 'yup', 'nope' )
    def sprintf(fmt, *args) ; nqp::sprintf(fmt, args) ; end
    puts sprintf("pid=%s time=%d", nqp::getpid, nqp::time_i)
```
this includes nqp control-flow functions:
```
    n = 99
    nqp::while n > 0, begin
        puts "#{n} green bottles standing on the wall"
        puts "#{n} green bottles standing on the wall"
        puts "and if one green bottle should accidently fall..."
        nqp::sleep 1
        n -= 1
        puts "there'd be #{n} green bottles standing on the wall"
        nqp::sleep 2
    end
```

Rubyish does a limited amout of context sensitive parsing, E.g.
```
   yy = 37
   puts yy -5   # parsed as an expression; output is 32

   def xx(n) ; 37 - n; end
   puts xx -5   # parsed as a function call; output is 42
```

This only works only if the functions and methods have been previously declared.

If in doubt, use parenthesis for function calls, `capitalize(name)` vs `capitalize name`; and make method calls explicit, `self.fibonacci(n - 1)` vs `fibonacci(n - 1)`.
