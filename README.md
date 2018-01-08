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
## License

Licensed under [EPL](http://www.eclipse.org/legal/epl-v10.html)

Copyright 2010, 2011, 2012, 2013 Hugo Duncan.
