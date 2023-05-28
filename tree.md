
# Component types

`cabbb068-bbcc-5c01-b115-35aba6f7ee07`


---

`91cfb2a7-ddb3-51bd-855f-0f2925150c79`

`TreeIndex`
Is a proxy for a `LListIndex` from `linked_list.d`, which is a proxy for a `ulong` array index. As with `LListIndex`, you can't add / subtract from a `TreeIndex` and get another guaranteed valid `TreeIndex`. Instead you either use linked list function to get the next internal linked list element, or tree functions to get child / parent nodes in the tree

---

`c403ebf7-e576-5736-b882-062fc6e76bda`

`mixin template NodeBody()`
This is the body of a `Node` which a `TreeBody` takes. Whatever is the `Node` in `TreeBody!Node` has to look just like this `NodyBody` or have mixed-in `NodeBody`

eg:

```D
struct MyNode {
  mixin NodeBody!();
  int value = 5;
}
```
Then you can use this node like:
```D
struct MyTree {
  mixin TreeBody!MyNode;
  ...
}
```

# Tree body

`963bcc1c-243f-586e-9417-b5ad5be412f3`

`mixin template TreeBody(Node)`
This is intended to be mixed into the body of a struct or class. The `Node` parameter should be something that looks like `NodeBody` or has mixed-in `NodeBody`

To use this, you just have to mix it into your own tree type:
```D
struct MyTree {
  mixin TreeBody!MyNode;
  ...
}
```

This provides a constructor `this(args...)` which makes a root node by sending the `args` parameter sequence to the `Node` default constructor

---

`a25bc4c5-984f-5b95-90e8-33595ad6a6bf`

Here are the trees properties:
* `TreeIndex rootIndex` -- The root index of the tree
* `hasRoot` -- Whether or not the tree actually has a root or not

Note that it isn't possible to determine if a root index is valid by itself, instead, check `hasRoot` to determine if you can use `rootIndex`

---

`e231b97e-a1ec-5c80-ae8c-4054d310fe1b`

`ref Node get(TreeIndex index)`

This is the typical way of getting a node from the tree

Should be $O(1)$

---

`d60d71bd-5dd6-5fc4-813a-25170e3520db`

`TreeIndex addRoot(args...)`

The very first node in the tree can either be made in the constructor (if one is used), or be made using `addRoot`, which takes a parameter sequence which it sends to the `Node` default constructor to make the root `Node`

It returns the root node's index as a `TreeIndex`

Should be $O(1)$

---

`50b35196-0fd3-56ab-8fdd-cbb1dfad36ee`

`TreeIndex appendChild(TreeIndex parentIndex, args...)`

`TreeIndex prependChild(TreeIndex parentIndex, args...)`

Makes a new `Node` by sending the `args` parameter sequence to the default `Node` constructor, and either `append`ing or `prepend`ing the new `Node` to the given `parentIndex`

They both return the new `Node`'s index as a `TreeIndex`

Note that `appendChild` should generally be $O(1)$, `prepend` child moves all elements in the parent's `childIndices` array every time, so `prependChild` is $O(c)$ where $c$ is the number of children the parent node has

---

`d69417f9-54cc-5bb8-9050-b494ae6f505d`

`TreeIndex parentIndex(TreeIndex childIndex)`

Returns the parent `TreeIndex` of the node at `childIndex`. ie: `tree.parentIndex(childIndex)` is `childIndex`'s parent location in the tree

And:

`d2ada34e-3a4d-5c16-8f64-e7a23906264f`

`TreeIndex childIndex(parentIndex, ulong i)`

Gets the `i`th child `TreeIndex` of the given parent node at `parentIndex`. 

Both of the above functions should be $O(1)$

---

`11288391-89be-5326-969a-701b5d7796ea`

`ref This disconnectChild(TreeIndex parentIndex, ulong i)`

Disconnect the `i`th child node from the parent node at `parentIndex`. Doesn't remove the child node from the tree. See `ref This remove(TreeIndex)` below to actually disconnect and remove nodes

And 

`f2aa63e4-b12b-5d48-9068-f6588636b58f`

`ref This disconnectChild(TreeIndex parentIndex, TreeIndex childIndex)`

Just like the other `disconnectChild` overload above, but the child index is referenced by `TreeIndex`. This necessitates a search for the right `parent.childIndices` array index, so the more children the parent has, the (very marginally) longer the function takes

Both of the above functions return references to the list itself, so they are chainable

---

`899c5147-5448-526b-b696-5f0714466af5`

`ref This remove(TreeIndex indexToRemove)`

Removes the node at the given index and the entire subtree rooted at that node

This returns a ref to the list itself, so this is chainable

This should be $O(c) + O(s)$ where $c$ is the number of children of the parent and $s$ is the size of the subtree rooted at the removed node

---

`0ee74879-1218-5a31-aba4-94f5e34412bd`

These redundantly are consistency and correctness checks that should be called in `version(assert)` after list modifications:

* `checkHasRoot` -- check `hasRoot` agrees with `length != 0`
* `checkRootIndex` -- check root index is within the array
* `checkNoChildIndexDuplicates` -- check that no node has a child included multiple times in its child indices array
* `checkChildParentSymmetry` -- checks each node's children have the node as their parent, and each node's parent has that node as a child
* `checkForCycles` -- check that no two nodes have the same child node

All of the above are called with:

`doPostModificationCheck()`

---

`b2ccdc0e-0e51-57d4-a065-ea75639efcdf`

`ref This applyDeepestChildrenFirst(TreeIndex startIndex, void delegate(TreeIndex) dg)`

Applies the loop body `dg` on each node depth first, which means: drill down to leaf nodes, apply `dg` to the deepest children, then when those children are done, do their parent, go up until there are more children, etc

ie, this order:
```
        7
      /   \
     /     \
    3       4
   / \     / \
  1   2   5   6
      
```
Where the 1 node is done first, 2 node node 2nd, etc

---

# Unit tests

`25e8f47d-8dba-5c73-b974-78d426726e43`

Look at these to see how to make use the library


