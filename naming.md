# Naming

# Donts

* Don't include external context in names or (don't name things based on how they are used).  Consider
  `loadimagesWhenStartingSystem`, if this function is now called when the
  system is not starting its name ceases to make sense and it will confuse
  readers of the code. Instead we can call the function `loadimages` now it can be used in any context and its name makes sense.
