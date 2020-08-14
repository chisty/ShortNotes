## Quick Sort

Quick Sort is widely used in every language. In C#, the method `Array.Sort() / List.Sort()` uses both *Insertion* Sort, *Quick* Sort, *Heap* Sort internally depending on below criteria:

- If the partition size is less than *16* , it uses *Insertion Sort.* The framework designers have chosen 16 since *imperically* 16 seems to speed up most cases without slowing down :p . [Reference](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/arraysorthelper.cs,4bf3d2825650d909)
- if the partition size exceeds 2 log n, where n is the range of input array, it uses *Heap Sort.*
- Otherwise, it uses *Quick Sort*.

So, basically the `Array.Sort() / List.Sort()` is a hybrid sort, it  selects different algorithm based on the partition size in runtime.



