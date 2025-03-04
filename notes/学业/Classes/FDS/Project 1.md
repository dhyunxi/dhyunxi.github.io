
# Introduction
---
- As an common procedure, searching plays an important role in our daily life. There are many ways to complement searching with different time and space complexity. In this project, we will do the search job in four algorithm, including iterative binary search, recursive binary search, iterative sequential search and recursive sequential search. We will analyze their complexity and compare their actual time consumption which would be a testament for their time complexity.
- In part 1, we would make a detailed account of our program, which have four functions,  corresponding to the four algorithms above. And we would implement the program with some data, including those could be found and those couldn't.
- In part 2, we would analyze the worst case complexity of each function.
- In part 3, we would test the four functions in the worst case and record them to form a table, with which we would see the difference between algorithms clearly.

# Part 1
---
- First and foremost, we assign the array A and initialize it.
```c++
#define MAX 10005
int A[MAX]; // the array where we would do the search 
void A_init(int N)
{
    //initialize the array A
    for(int i=0;i<N;i++)
    {
        A[i]=i;
    }
}
```
## Sequential Search
### Iterative Sequential Search
- In the iterative sequential search, we use a loop to traverse the elements in the array to check if it's matched.
- The function would require an integer as a target, and return an integer as the index of the target, which would be -1 if the target is not found.
```c++
// Sequential Search of iterative version
int iterative_sequential_search(int target)
{
    //search the target in a loop
	for(int i=0;i<N;i++)
	{
		if(A[i]==target){   //check if there's a match
			return i;       //if matched, then return the index
		}
	}
	return -1;              //return -1 when the target couldn't be found
}
```
### Recursive Sequential Search
- In the recursive sequential search, we use a recursion as a substitution of iteration, which means if we failed to match the target with the A[i], then we would do the search again in A starting with A[i+1].
- To this end, the function needs two arguments, which are the target number and the starting index.
- It's obvious that the recursion doesn't help much.
```c++
// Sequential Search of recursive version
int recursive_sequential_search(int target,int start)
{
    if(start>=N) return -1; //check if the index is out of bounds
    if(A[start]==target)    //if matched, then return the index
        return start;
    else                    //else, do the recursion
        return recursive_sequential_search(target,start+1);
}
```

## Binary Search
### Iterative Binary Search
- In the iterative binary search, we use variables `ll` and `rr` to mark the left and the right bounds of our search area. We use a loop to adjust the bounds of interval, which would ends when we find the target or the whole array is searched.
```c++
// Binary Search of iterative version
int iterative_binary_search(int target)
{
    int ll=0,rr=N-1;    //initialize the left and right bounds
    int mid=(ll+rr)/2;
    while(ll<=rr){
        if(A[mid]==target)  //matched
            return mid; 
        else if(A[mid]<target)  //if less, search in the right part
        {
            ll=mid+1;
    }else{                      //if more, search in the left part
            rr=mid-1;
        }
    }
    return -1;
}
```


### Recursive Binary Search
- Similar to the iterative version, we use `ll` and `rr` to mark the bounds and do the recursion when the match failed.
```c++
// Binary Search of recursive version
int recursive_binary_search(int target,int ll,int rr)
{
    if(ll>=rr)          //legitimacy judgement
        return -1;
    int mid=(ll+rr)/2;
    if(A[mid]==target) 
        return mid;
    else if(A[mid]<target)
        return recursive_binary_search(target,mid+1,rr);
    else
        return recursive_binary_search(target,ll,mid-1);
}
```

## Implementation
- To test the feasibility of our functions, we would set the N as 100 with the target as 1 and 100, as the number 1 could be found and 100 couldn't.
```c++
int main()
{
	int N=100;
	int target_a=1,target_b=100;
	A_init(N);
	
}
```