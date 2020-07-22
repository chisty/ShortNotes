**Problem Map**
------------
Here, I will try to map problem patterns to specific or closer strategies. This will help to find right approach to solve the problem.  Since there is always a finite set of problem patterns, we can easily group them. 

[Here,](https://medium.com/better-programming/the-ultimate-strategy-to-preparing-for-the-coding-interview-ee9f7eb439f3 "Here,") is a very good article related to this. **Must Read!**

##### Ad-Hoc / Observation
------------
- If any brute force solution exists with O(n2) time and O(1) space, there must be 2 other solutions: (1)- O(n) time & O(n) space using a Map/Set/Dictionary. (2)- O(n logn) time & O(1) space using sorting.
- If the problem involves a LinkedList and we can't use extra space, then we should use fast & slow pointer approach.



##### Search/ Find
------------
- If the input is *sorted*, then think about *Binary Search, Two pointers *etc.
- If we need to find *top/ maximum/ minimum / closest *K elements among N elements, then think of *Heap*.
- To find all *combination/permutation*, think of recursive Backtracking or *iterative BFS.*
- Most of the Tree/Graph problem can be solved using BFS/DFS.
- Any recursion solution can be converted to an iterative approach using a Stack.



##### Dynamic Programming
------------
- If the problem is asking for optimization, we will need DP to solve it.
- Memoization is a good optimization technique.
- To find a common substring among a set of strings, we need to use HashMap or a Trie.
- To search among a bunch of strings, Trie is the best data structure.
- 
