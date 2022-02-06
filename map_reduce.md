## Map Reduce In System Design

**1. Paper**  
[MapReduce](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf).  
**Focus on the 'Figure 1. Execution overview'** in this page.

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
