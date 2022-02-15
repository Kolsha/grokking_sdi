## Bloom Filter

**1. Background/Problem**  
Let's say we have a large set of structured data(identified by record IDs) stored in a set of data files. What's the most efficient way to know which file might contain our required data (**false or true**) ?

**2. Solution: Bloom Filter**
- Bloom Filter is a **data structure**;
- Bloom Filter can be used to tells **whether an element may be in a set (i.e. false positive), or definitely is not in a set**.
- Initialization of a Bloom Filter:
    - a bit-array of **m** bits;
    - all bits are set to **0**.
- Add an new element into a Bloom Filter:
    - there are **k** predefined hash functions; The output range of these hash functions is **[0, m]**;
    - for an new element, feed it to these **k** hash functions. Therefore, we will get **k** hash values.
    - for every **value** in the output **k** hash values, mark **bitarray[value] = 1**.
    - **time complexity = O(k) which is constant time**
- Decide whether an element may be in a set ?
    -  for this element, feed it to these **k** hash functions. Therefore, we will get **k** hash values.
    -  for every **value** in the output **k** hash values, if there is any value which satisfies **bitarray[value] = 0**, this element must not be in this set. Else, it may be in this set.
    -  **time complexity: O(k) which is constant time**.
