## 1.说一下几种常见的排序算法和分别的复杂度。

| 排序算法 | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 稳定性 |
| :--- | :--- | :--- | :--- | :--- |
| 冒泡排序 | O\(n²\) | O\(n²\) | O\(1\) | 稳定 |
| 选择排序 | O\(n²\) | O\(n²\) | O\(1\) | 稳定 |
| 插入排序 | O\(n²\) | O\(n²\) | O\(n²\) | 稳定 |
| 快速排序 | O\(n\*log2n\) | O\(n²\) | O\(log2n\)~O\(n\) | 不稳定 |

## 2.用Java写一个冒泡排序算法

```
public void bubbleSort(int[] a) {
  if (a == null || a.length < 2) {
    return;
  }
  for (int i = a.length - 1; i > 0; i--) {
    for (int j = 0; j < i; j++) {
      if (a[j + 1] < a[j]) {
        int temp = a[j + 1];
        a[j + 1] =  a[j];
        a[j] = temp;
      }
    }
  }
}
```

## 3.描述一下链式存储结构。

1、比顺序存储结构的存储密度小\(链式存储结构中每个结点都由数据域与指针域两部分组成，相比顺序存储结构增加了存储空间\)。

2、逻辑上相邻的节点物理上不必相邻。

3、插入、删除灵活 \(不必移动节点，只要改变节点中的指针\)。

4、查找结点时链式存储要比顺序存储慢。

5、每个结点是由数据域和指针域组成。

6、由于簇是随机分配的，这也使数据删除后覆盖几率降低，恢复可能提高。

## 4.如何遍历一棵二叉树？

```
List<List<Integer>> print(TreeNode root) {
  ArrayList<ArrayList<Integer>> ll = new ArrayList<>();
  if (root == null) {
    return ll;
  }
  ArrayList<Integer> l = new ArrayList<Integer>();
  TreeNode p = root;
  TreeNode last = null;
  LinkedList<TreeNode> queue = new LinkedList<TreeNode>();
  queue.add(root);
  while (!queue.isEmpty()) {
    TreeNode node = queue.poll();
    l.add(node.val);
    if (node.left != null) {
      last = node.left;
      queue.add(node.left);
    }
    if (node.right != null) {
      last = node.right;
      queue.add(node.right);
    }
    if (p == node) {
      p = last;
      ll.add(l);
      l = new ArrayList<>();
    }
  }
  return ll;
}
```

## 5.倒排一个LinkedList。

```
Collections.reverse(linkedList);
```

倒序链表可以借助Stack类或者使用递归

```
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
    Stack<Integer> stack = new Stack<Integer>();
    while (listNode != null) {
        stack.push(listNode.val);
        listNode = listNode.next;
    }
    ArrayList<Integer> list = new ArrayList<Integer>();
    while (!stack.isEmpty()) {
        list.add(stack.pop());
    }
    return list;
}
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
    ArrayList<Integer> list = new ArrayList<Integer>();
    ListNode pNode = listNode;
    if (pNode != null) {
        if (pNode.next != null) {
            list = printListFromTailToHead(pNode.next);
        }
        list.add(pNode.val);
    }
    return list;
}
```

## 6.用Java写一个递归遍历目录下面的所有文件。

```
void listAll(File directory) {
    if (!(directory.exists() && directory.isDirectory())) {
        throw new RuntimeException("目录不存在");
    }
    File[] files = directory.listFiles();
    for (File file : files) {
        System.out.println(file.getPath() + file.getName());
        if (file.isDirectory()) {
            listAll(file);
        }
    }
}
```



