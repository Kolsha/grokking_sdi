## Bloom Filter

**Background/Problem**  
Let's say we have a large set of structured data(identified by record IDs) stored in a set of data files. What's the most efficient way to know which file might contain our required data (**false or true**) ?

**Solution: Bloom Filter**
- Bloom Filter is a **data structure**;
- Bloom Filter can be used to tells **whether an element may be in a set (i.e. false positive), or definitely is not in a set**.
