# Laicode 645. Deque By Three Stacks

用三个栈来模拟实现双端队列。

那么一个栈`left`用来存放在队列头部的元素，一个栈`right`用来存队列尾部的元素，`buffer`用来做缓冲。

`offerFirst`和`offerLast`就正常压入`left`和`right`。

`poll`和`peek`就返回各自的栈顶就好，但是这里需要考虑栈为空的情况。如果`peekFirst`的时候，`left`为空的话，就得返回`right`的栈底元素。所以我们需要将两个栈尽可能的均衡一下，当一方为空时，另一方挪一半的元素过去。

比如`peekFirst`且`left`为空的时候，我们想把`right`靠近栈底的那部分挪到`left`，并且让原`right`的栈底元素变成`left`的栈顶元素。其实这里由于栈的性质，从`right`往`left`挪的时候，`right`的栈底会自动变成`left`的栈顶。但是呢，`right`靠近栈顶的那一半，不能乱动。所以我们用一个`buffer`来先倒序存放那一半，挪完之后再压回去，顺序就和原来还是一样。如此一来，我们便实现了挪一半过去的操作，并且挪过去的部分栈底变栈顶，留下来的部分不变。

我们可以看到，只有在对`left`和`right`做均衡的时候，才会出现挪动元素的情况。挪动一半的复杂度是O(N)，但是一般情况下并不会次次都要均衡。

假设我们现在整个双端队列有N个元素，`left`为空，元素全在`right`。我们需要`pollFirst`的话，就需要挪`N/2`个元素到`left`。再做`N/2`次`pollFirst`才会再次耗尽`left`从而需要再次均衡元素，届时只需要挪`N/4`个元素。再做`N/4`次`pollFirst`……

所以其实均摊下来的各个操作实现复杂度，其实就是常数级了，也就是O(1)。

```java
public class Solution {
  Deque<Integer> left, right ,buffer;
  public Solution() {
    // Write your solution here.
    left = new ArrayDeque<>();
    right = new ArrayDeque<>();
    buffer = new ArrayDeque<>();

  }
  
  public void offerFirst(int element) {
    left.push(element);
  }
  
  public void offerLast(int element) {
    right.push(element);
  }
  
  public Integer pollFirst() {
    move(right, left);
    return left.isEmpty() ? null : left.pollFirst();
  }
  
  public Integer pollLast() {
    move(left, right);
    return right.isEmpty() ? null : right.pollFirst();
  }
  
  public Integer peekFirst() {
    move(right, left);
    return left.isEmpty()? null : left.peekFirst();
  }
  
  public Integer peekLast() {
    move(left, right);
    return right.isEmpty()? null : right.peekFirst();
  }
  
  public int size() {
    return left.size() + right.size();
  }
  
  public boolean isEmpty() {
    return left.isEmpty() && right.isEmpty();
  }

//when the destination stack is empty, move half of the elements from the source stack to the dest stack
  private void move(Deque<Integer> src, Deque<Integer> dest){
    if (!dest.isEmpty()){
      return;
    }
    int halfSize = src.size() / 2;
    for(int i = 0; i < halfSize; i++){
      buffer.offerFirst(src.pollFirst());   //src: 1 2 3 4 5 6 ,   buffer: 3 2 1    
    }
    while(!src.isEmpty()){
      dest.offerFirst(src.pollFirst());  // src 4 5 6, dest 6 5 4
    }

    while(!buffer.isEmpty()){
      src.offerFirst(buffer.pollFirst()); // buffer 3 2 1 ,  src  1 2 3  
    }

  }
}
```