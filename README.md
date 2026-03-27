# block

A Scheme block/return-from style library

This will eventually be a R7RS library for a CL-style `block`/`return-from` macro.

```scheme
(define-syntax block
  (syntax-rules ()
    ((block name expr ...)
     (call-with-current-continuation
       (lambda (name) expr ...)))))
```

And maybe a version that supports returning multiple values?

```scheme
(define-syntax block-values
  (syntax-rules ()
    ((block-values name expr ...)
     (apply values
       (call-with-current-continuation
         (lambda (%k)
           (define (name . args) (%k args))
           (list (begin expr ...))))))))
```

This can be combined with `dynamic-wind`.

```scheme
(block exit
  (dynamic-wind
    (lambda () (display "acquire resource\n"))
    (lambda ()
      (exit 42)                        ; escape
      (display "never reached\n"))
    (lambda () (display "release resource\n"))))  ; fires on escape
```

Using `call/ec` would be better, if available. A `cond-expand` could be used for that, but I'll need to find implementations that support it and test with them.

```scheme
(cond-expand
  (chibi
    (import (chibi))
    (define call/ec-impl call-with-escape-continuation))
  (chezscheme
    (define call/ec-impl call-with-escape-continuation))
  (racket
    (define call/ec-impl call-with-escape-continuation))
  (else
    (define call/ec-impl call-with-current-continuation)))
```
