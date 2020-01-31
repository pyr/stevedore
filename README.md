stevedore: An embedding of shell script in clojure.
===================================================

[![Build Status](https://secure.travis-ci.org/pyr/stevedore.png)](http://travis-ci.org/pyr/stevedore)

This is a resurrection of the script abstraction part of https://github.com/pallet/stevedore.
This only supports bash as a target for now.


```clj
:dependencies [[spootnik/stevedore "0.9.1"]]
```

## Usage

```clojure
(require '[stevedore.bash :as bash])

(bash/script
  (defn showfiles [pattern] (pipe (ls) (grep @pattern)))
  (showfiles foo))
```

## The stevedore DSL

### Simple expressions

At its core, stevedore provides a way to write shell scripts with
sexprs with minimal fuss.  All non-special symbols are directly
converted to their string representation in the output.

```clojure
(bash/script
  (systemctl restart nginx))
  
;; 
;; systemctl restart nginx
;;
```

Forms that cannot be expressed as symbols can be quoted, and keywords
will be converted with `name`:

```clojure
(bash/script
  (ls "/home")
  (apropos :nginx))
  
;; 
;; ls /home
;; apropos nginx
;;
```

### Variables

Variables can be defined either with `defvar` or `set!`

```clojure
(defvar tmpdir "/tmp")
(set! cachedir "/tmp/cache")

;;
;; tmpdir=/tmp
;; cachedir=/tmp/cache
```

Variables (environment variables, or local ones defined with `defvar` or `set`) can
be looked up with `deref` or the `@` reader literal:

```
(echo @tmpdir)
(echo (deref cachedir))

;;
;; echo ${tmpdir}
;; echo ${cachedir}
```

`deref` accepts additional optional arguments: 

- `:default` for the `${foo-bar}`
- `:default-value` for `${foo:-bar}`
- `:default-assign` for `${foo=bar}`
- `:default-assign-value` for `${foo:=bar}`
- `:alternate` for `${foo+bar}`
- `:alternate-value` for `${foo:+bar}`
- `:error` for `${foo?bar}`
- `:error-value` for `${foo?-bar}`

If you are unfamiliar with these bashisms, please refer to
https://www.tldp.org/LDP/abs/html/parameter-substitution.html

### Pipes

```clojure
(pipe 
  (cat "/etc/passwd")
  (grep nobody)
  (wc -c))
  
;; cat /etc/passwd | grep nobody | wc -c  
```

### Boolean expressions

Beware, both `or` and `and` silently drop forms beyond the second one

```clojure
(or (first expr) (second expr))
(and (first expr) (second expr))

;; first expr || second expr
;; first expr && second expr
```

### Conditional expressions

The understood stanzas are: `if`, `if-not`, `when`, `when-not`, and `case`:

```clojure
(if (some expr)
   (truthy expr)
   (falsy expr))
   
(when (some expr)
  (first expr)
  (second expr))
```

`case` maps to a bash case expression, there is no support inferring default,
but `*` can be used as with bash case expressions:

```clojure
(case (some expr)
  first-val first-expr
  second-val second-expr
  * default-expr)
```

### Loops

```clojure
(while (some-condition)
  (first expr)
  (second expr))
```

```clojure
(doseq [x @(cat /etc/passwd)]
  (echo x))
```

### Functions

```clojure
(defn greet [firstname lastname]
  (echo "hello" @firstname @lastname))
  
(hello "Hugo" "Duncan")
```


## License

Licensed under [EPL](http://www.eclipse.org/legal/epl-v10.html)

Copyright 2010, 2011, 2012, 2013 Hugo Duncan.
