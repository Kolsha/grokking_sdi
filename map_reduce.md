## Map Reduce In System Design

**1. The Paper Published by Jeff Dean on MapReduce**  
In this paper [MapReduce](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf), an execution overview figure is presented. One thing to note in the execution overview:  
- When the reducer worker has read all intermediate data, it **sorts** it by intermediate keys so that **all occurrences of the same key are grouped together**.

**2. What is Map Reduce ?**
- Map
    - input: _a_ key/value pair
    - output: _a set of_ intermediate key/value pair_s_.
- Reduce
    - input: an intermediate key, and _a set of values for that key_.
    - output: 0 or 1 output value.

```
map(String key, String value):
    // key: document name
    // value: document contents
    for each word w in value:
        EmitIntermediate(w, "1");

reduce(String key, Iterator values):
    // key: a word
    // values: a list of counts
    int result = 0;
    for each v in values:
        result += ParseInt(v);
    Emit(AsString(result));
```
