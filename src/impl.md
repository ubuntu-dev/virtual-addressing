* **lifetimes** controlling lifetimes of vaddrs
  * **construct** creating new (empty-ish) vaddrs
  * **destruct** demolishing vaddr arrays
* **ctypes** raw C pointers to and from virtual addressing
  * **ctypes_in** transforming C types to vaddrs
  * **ctypes_out** transforming vaddrs to C types
* **locations** indexing and searching
  * **searching** get index, given value constraints
  * **indexing** get value, given index
* **namespaces** function name organisation
  * **ns_construct** creating namespace function structs
  * **ns_destruct** destroying namespace function structs
* **mutations**
  * **modifying** by-value inserting, deleting, extending, changing
  * **transforming** sorting, reversing, mapping, filtering, reducing
* **attributes** convert / interpret properties of vaddrs
  * **ranges** conversions to / from range notation
  * **metadata** get/set metadata, flags etc
* **visualising** string representations of vaddrs
  * **repr_in** creating vaddrs from string / human data
  * **repr_out** displaying vaddrs as human-readable

a vaddr (virtual address object) is a two-dimensional array. its elements are "triples", which are 2 or 3-element arrays of `uint64_t`.

when R (explicit range notation) is not enabled the structure is
```
A = [
  [L S R] - L: number of triples following S: spoofing virtual length R: using range notation
  [V Z 0] - V: a non-zero value Z: the number of zeroes preceding V 0: zero (unused field)
  [V Z 0] - ...
  ...
  NULL  - a null terminator, for simpler iteration.
]
```

when R (explicit range notation) is enabled the structure is
```
A = [
  [L S R] - L: number of triples following S: spoofing virtual length R: using range notation
  [V B T] - V: a non-zero value B: the bottom index of zero range T: the top index of zero range
  [V B T] - ...
  ...
  NULL  - a null terminator, for simpler iteration.
]
```

on x86-64, the memory used by either structure for N non-zero values is

```
( 8 * 3 * (N + 1) ) + 8
 (1) (2)      (3)    (4)
```
1. `sizeof (uint64_t)`
2. the length of each triple
3. the initial metadata triple element (containing `[L S R]`)
4. an extra pointer element for `NULL`

for 2 non-zero values, the result is 80 bytes.
2 values for 80 bytes seems rich but here if there are more than 10 zeroes around / between the non-zero values, then memory is saved.

if there were 2 000 non-zero values the memory used would be 48 032 bytes but if there are more than 6 004 zeroes around the non-zero elements, then memory will be saved.

the case in which there are many "empty" runs of zeroes and a non-zero value once in a while is exactly what is found in implementing hashtables through raw C arrays.
