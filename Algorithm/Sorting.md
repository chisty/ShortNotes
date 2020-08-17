## Quick Sort

Quick Sort is widely used in every language. In C#, the method `Array.Sort() / List.Sort()` uses both *Insertion* Sort, *Quick* Sort, *Heap* Sort internally depending on below criteria:

- If the partition size is less than *16* , it uses *Insertion Sort.* The framework designers have chosen 16 since *imperically* 16 seems to speed up most cases without slowing down :p . [Reference](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/arraysorthelper.cs,4bf3d2825650d909)
- if the partition size exceeds 2 log n, where n is the range of input array, it uses *Heap Sort.*
- Otherwise, it uses *Quick Sort*.

So, basically the `Array.Sort() / List.Sort()` is a hybrid sort, it  selects different algorithm based on the partition size in runtime.

I have found `Golang` also does kind of same, it also use *Shell Sort* when partition is less than 12. Here is the [reference](https://golang.org/src/sort/sort.go).

### Theory

Quick sort uses *divide & conquer* technique. It selects a *pivot* item from the list and partitions the list into *two sub list* based on the pivot. It loops through the list and checks if each item is *less than or greater than* the pivot; if item is less than pivot item, it inserts into left sub list, and for greater item inserts into right sub list. By doing this, we can find the *right place for the pivot*, which is between the left and right sub list, the *exact* position. 

Now, we can recursively do same thing for each sub list. Then 2 other pivot can be selected from both sub list, and eventually each list will be sorted. This sorting can be done in place so it is very fast. 

From Wikipedia, The space complexity is at most *O(log n)*. Because, it needs to store the information for nested recursive calls. log n, is practically very small footprint. For example, 1 billion int will require 4GB memory, but for recursive call it will take at most 30 stack frames. If each stack is 40 bytes, it will take 1200 byte or 1.2 KB in total.

### Implementation

```c#
//Approach 1, by selecting the first item as pivot.
private List<int> Sort(List<int> temp)
{
    if (temp.Count == 0) return temp;
    
    var left = new List<int>();
    var right = new List<int>();
    var pivot = temp[0];
    for (var i = 1; i < temp.Count; i++)	//loop through each element.
    {
        if (temp[i] < pivot)	    left.Add(temp[i]);	        //if item is less than pivot, insert into left
        else		            right.Add(temp[i]);         //if item is greater, insert into right
    }

    left = Sort(left);	            //now try to sort the left sub list 
    left.Add(pivot);   	            //add pivot after the left sorted list, all left item is less than pivot.
    left.AddRange(Sort(right));     //now try to sort the right sub list, and append the sorted list.
    return left;
}

public void Test()
{
    var data = new List<int> { 9, 2, 5, 6, 4, 5, 3, 7, 10, 10, 1, 12, 8, 11, 10 };
    var result = Sort(data);  // result is sorted here.
}
```

In the above approach, we are selecting the first element as pivot. But here we have created new list to hold the left and right elements. Below is a more efficient approach where we used the same list/memory to do the sorting:

```c#
private int GetPivotByPartition(int[] data, int start, int end)
{
    int pivot = data[end], index = start, temp;

    /* We will use index to place the elements which is smaller than pivot. Set index equal to start. So, loop through each item,
    if item is less than pivot, put it in the position pointed by index (basically which is the first position since index 
    is 0 at first). Then increment index. when the loop completes, we know that all the item before current index position
    is lower than the pivot. Now, place pivot at index position. Next, we can do recursive call on left partition and right
    partition based on pivot position. Eventually, full list will be sorted. */
    
    for (int i = start; i < end; i++)
    {
        if (data[i] <= pivot)
        {
            temp = data[i];
            data[i] = data[index];
            data[index] = temp;
            index++;
        }
    }
    temp = data[end];	            //here data[end] is basically pivot item.
    data[end] = data[index];	    //swap data[end]/pivot with index.
    data[index] = temp;
    return index;
}

private void SortByPartition(int[] data, int start, int end)
{
    if (start >= end) return;

    var pivot = GetPivotByPartition(data, start, end);	
    SortByPartition(data, start, pivot - 1);	//Sort left side of the pivot
    SortByPartition(data, pivot + 1, end);		//Sort right side of the pivot
}

public void Test()
{
    var data = new List<int> { 9, 2, 5, 6, 4, 5, 3, 7, 10, 10, 1, 12, 8, 11, 10 };
    var input = data.ToArray();
    SortByPartition(input, 0, data.Count - 1);  //input is sorted here.
}
```

