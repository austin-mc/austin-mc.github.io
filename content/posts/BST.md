---
title: "Binary Search Trees"
date: 2022-08-10
draft: false
author: "Austin Christiansen"
authorLink: "https://austinchristiansen.com"
description: ""
---

<!--more-->

### Creating a header-only templated Binary Search Tree in C++.


In this post, we'll explore how to make a Binary Search Tree that is compatible with a variety of data types in C++.

&nbsp;

In this post, I create a templated Binary Search Tree in C++. Templated data structures allow the programmer to use the class with any data type that can be compared with the standard comparison operators. This flexibility allows one class to be used for multiple data structures without creating an entire new data structure.

&nbsp;

[Check out the GitHub repository with the code used in this article](https://github.com/austin-mc/BinarySearch)

&nbsp;

#### Binary Search Tree Basics

If you're already familiar with Binary Search Trees, feel free to skip to the next section.

A Binary Search Tree is just a Binary Tree with a couple extra properties:
1. All values in the left subtree are less than the value of the parent
2. All values in the right subtree are greater than the value of the parent

&nbsp;

![Binary Search Tree Visualization](/images/BST.png)

&nbsp;

These properties make Binary Search Trees extremely useful for storing data that will need to be searched frequently. To find a value in the tree, we simply compare it to the root node. If the value of the root is greater than the value we want to find, we move left in the tree. If its less than the value we want to find, we move right.

#### Template Basics
Templating in C++ is useful for creating data structures that may be used for a variety of data types. To template our class, we need to let C++ know by putting `template <typename T>` or `template <class T>` on the line before the class declaration and **every** method using a templated type. When we define a method, we'll add a `<T>` to the name of the class as well. Here's an example constructor for the BST class:

&nbsp;

```cpp
//Constructor with initial value for root
template <typename T>
BST<T>::BST(T data) {
    root = new Node<T>(data);
}
```

&nbsp;


Notice how our parameter for the constructor is of type `T` as well. We'll use this through the code as a placeholder for whatever data type the BST will actually hold when its used. One last thing to keep in mind when creating a templated data structure is that they are typically created as header-only libraries. This means that the class definition and all method definitions will be contained in a single header file instead of the typical .h and .cpp file combination. 


#### Defining The Node

Now that we have the basics, lets get started on our BST. The first thing we'll do is define our nodes. A node is a basic data structure that holds our data as well as pointers to the left and right leaf nodes. In C++, this can be done with either a struct or a class. In this example I chose to use a struct. 

&nbsp;

```cpp
template <typename T>
struct Node {
    T data;
    Node* left;
    Node* right;
    Node(T data);
};

// Constructor
template <typename T>
Node<T>::Node(T data){
        this->data = data;
        this->left = nullptr;
        this->right = nullptr;
}
```

&nbsp;

The only method in our node struct is a constructor. Don't forget the `template <typename T>` prior to the method and the `<T>` before the scope resolution when defining the method.

#### Binary Search Tree Basic Methods

When creating our BST, theres a few methods we'll want to define:
1. Constructor
2. Destructor
3. Insert
4. Find
5. Remove
6. Empty
7. Print

&nbsp;

We'll begin by declaring the class and method footprints. In this example I created public methods and private helper methods to make code cleaner when using the BST. This allows the user to call methods in a more natural way, such as `bst.Insert(20)` as opposed to `bst.Insert(root, 20)`. This is purely personal preference and these methods can be written without helpers just as easily. 

&nbsp;

```cpp
template <typename T>
class BST {
    public:
        Node<T>* root;
        
        BST();
        BST(T data);
        ~BST();
        
        Node<T>* Insert(T data);
        Node<T>* Find(T data);
        Node<T>* Remove(T data);
        void Empty();
        void Print();
        
    private:
        Node<T>* Insert(Node<T>* root, T data);
        Node<T>* Find(Node<T>* root, T data);
        Node<T>* Remove(Node<T>* root, T data);
        Node<T>* FindPredecessor(Node<T>* root);
        void Print(Node<T>* root);
        void Empty(Node<T>* root);
};
```

&nbsp;

Now that we have the forward declarations completed for the methods, it's time to start defining the constructors and destructor. Since each node will be created using the `new` keyword, we'll need to make sure to have a destructor that can `delete` each of them when we're done with the BST to avoid a memory leak. We can easily do this by calling the `Empty` method that we'll define later on. 

&nbsp;

```cpp
//Default constructor
template <typename T>
BST<T>::BST() {
    root = nullptr;
}

//Constructor with initial value for root
template <typename T>
BST<T>::BST(T data) {
    root = new Node<T>(data);
}

//Destructor
template <typename T>
BST<T>::~BST() {
    Empty();
}
```

&nbsp;

#### Searching

The first method we'll look at is searching. Using recursion and the properties of Binary Search Trees, we can quickly find values in our tree. The recursive method will have 2 base cases: the current node is a `nullptr`, meaning the value doesn't exist in the tree, or the current node contains the value. If the value isfound, we return the pointer to the node containing the value. If the value isn't found, we'll return a `nullptr` to let the program know the value doesn't exist. This case will need to be handled each time the Find method is called. To recursively find the value, we compare the current data in the current node to the value being searched for. If the data in the current node is less than the value being searched for we recursively call the Find method with the left leaf node, and if its greater than the value being searched for we call the Find method with the right leaf node.

&nbsp;

```cpp
//Public Find method. Returns a nullptr if value isn't found.
template <typename T>
Node<T>* BST<T>::Find(T data) {
    return Find(root, data);
}

//Private helper method
template <typename T>
Node<T>* BST<T>::Find(Node<T>* root, T data) {
    if (root == nullptr) {
      //Value was not found in the tree
      return nullptr;
    }
    
    if (root->data == data) {
        return root;
    } else if (root->data > data) {
        return Find(root->left, data);
    } else {
        return Find(root->right, data);
    }
}
```

&nbsp;

#### Insertion

Inserting data into a Binary Search Tree is only slightly more complicated than searching for a value. In this example, the Insert method will return a pointer to the inserted node, but if this isn't needed the methods can return void or a boolean indicating a successful insertion. The base case for our recursion will either be when the current node is a `nullptr`, meaning that the node is empty, or when the data in the current node is equal to the data to be inserted. In the first case, we'll instantiate a new node containing our data and return it. In the second case, we'll return the node that already exists with our value. To get to our base cases, we will take advantage of the BST properties in the same way as we did when searching.

&nbsp;

```cpp
//Public Insert method
template <typename T>
Node<T>* BST<T>::Insert(T data) {
    return Insert(root, data);
}

//Private helper method
template <typename T>
Node<T>* BST<T>::Insert(Node<T>* root, T data) {
    if (root == nullptr) {
        root = new Node<T>(data);
        return root;
    }  else if (root->data == data) {
        //Already exists, return the existing object
        return root;
    } else if (root->data > data) {
        //Root data is greater than data to be inserted, so recursively insert on the left leaf
        root->left = Insert(root->left, data);
    } else{
        //Root data is less than data to be inserted, so recursively insert on the right leaf
        root->right =  Insert(root->right, data);
    }
    return root;
}
```

&nbsp;

#### Removal

Removing a node is perhaps the most complicated algorithm for a Binary Search Tree. When removing a node, there are three main cases:
1. The node to be removed has no children (easiest)
2. The node to be removed has one child
3. The node to be removed has two children (most complicated)

Below is the full code for removing a value from the BST. Included is a FindPredecessor method that will be used in the third case (this will be discussed in more detail shortly).

&nbsp;

```cpp
//Public Remove method
template <typename T>
Node<T>* BST<T>::Remove(T data) {
    root = Remove(root, data);
    return root;
}

//Private helper method
template <typename T>
Node<T>* BST<T>::Remove(Node<T>* root, T data) {
    if (root == nullptr) {
        return nullptr;
    }
    
    //Recusively look for the node to be deleted
    if (root->data > data) {
        root->left = Remove(root->left, data);
    } else if (root->data < data) {
        root->right = Remove(root->right, data);
    } else {
        //Node was found
        //3 cases to handle now: node is a leaf, node has one child, and node has 2 children
        
        //Node is a leaf:
        if(root->left == nullptr && root->right == nullptr) {
            delete root;
            return nullptr;
        //Node has one child:
        } else if (root->right == nullptr) {
            Node<T>* temp = root->left;
            delete root;
            return temp;
        } else if (root->left == nullptr) {
            Node<T>* temp = root->right;
            delete root;
            return temp;
        //Node has two children:
        } else {
            Node<T>* pred = FindPredecessor(root->left);
            root->data = pred->data;
            root->left = Remove(root->left, pred->data);
        }
    }
    return root;
}

//Find the right most node in the left subtree
template <typename T>
Node<T>* BST<T>::FindPredecessor(Node<T>* root) {
    if (root->right == nullptr) {
        return root;
    } else {
        return FindPredecessor(root->right);
    }
}
```

&nbsp;

Now that we've seen the complete code, let's break it down into smaller pieces. The first part of the method looks very similar to the Find method we wrote earlier, with one change. When a node is deleted, we need to update the parent nodes pointer so that it doesn't point to a deleted object. By assigning root->left or root->right to the corresponding recursive Remove call, we can make sure all the pointers are going to the correct objects and keep our tree intact. 

&nbsp;

```cpp
if (root == nullptr) {
  //Node wasn't found, return a nullptr
    return nullptr;
}
  
//Recusively look for the node to be deleted
if (root->data > data) {
    root->left = Remove(root->left, data);
} else if (root->data < data) {
    root->right = Remove(root->right, data);
} else {
  // node is found, do stuff ...
```

&nbsp;

###### Case 1: Node Has No Children
Now that we've found the node we want to delete, we must determine which of the three cases this node falls under. If the node has no children, we just need to remove the node and return a `nullptr` to the parent. This is the easiest of the three cases. The two images below show an example tree before and after a leaf node is removed.

![Tree before leaf node removal](/images/BSTRemove11.png)
![Tree after leaf node removal](/images/BSTRemove12.png)

&nbsp;

Below is the code removing a leaf node:


```cpp
//Node is a leaf:
if (root->left == nullptr && root->right == nullptr) {
    delete root;
    return nullptr;
}
```

&nbsp;

###### Case 2: Node Has One Child

For the second case, removal gets slightly more complicated. We must now replace the node we delete with its child node. To accomplish this, we can store the pointer to the child in a temporary variable, delete the current node, and return the pointer to the child back to the original parent node. The two images below show an example tree before and after the node is removed.

![Tree before node removal](/images/BSTRemove21.png)
![Tree after node removal](/images/BSTRemove22.png)

&nbsp;

Below is the code for this case:

```cpp
//Node has one child:
} else if (root->right == nullptr) {
    Node<T>* temp = root->left;
    delete root;
    return temp;
} else if (root->left == nullptr) {
    Node<T>* temp = root->right;
    delete root;
    return temp;
}
```

&nbsp;

###### Case 3: Node Has Two Children

The final case is the most complicated of the three. When the node has two children, we must find the proper value to replace it with. The first thing that may come to mind is to use one of the children, however in many cases this will violate the properties of a BST by having values in improper locations. What we need to do is replace the value with either the next smallest value or the next largest. Considering the properties of a BST, we know that the next smallest value will be in the left subtree and as far right in that subtree as we can go. This node is called the *predecessor*. The next largest value is in the right subtree of the node we want to remove and as far left in that subtree as we can go. This is called the *ancestor*. Either value is perfectly fine to use, but in this example I used the predecessor. 

The two images below show an example of what a tree will look like before and after removing a node with two children:

![Tree before node removal](/images/BSTRemove31.png)
![Tree after node removal](/images/BSTRemove32.png)

&nbsp;

Let's take a look at the FindPredecessor method. When we initially call this method, we'll call it on `root->left` to get started searching the left subtree. Then, the method will recursively go to the right until reaching the base case when the current node does not have a right leaf. We've reached the predecessor and will return a pointer to this node. 

&nbsp;

```cpp
//Find the right most node in the left subtree
template <typename T>
Node<T>* BST<T>::FindPredecessor(Node<T>* root) {
    if (root->right == nullptr) {
        return root;
    } else {
        return FindPredecessor(root->right);
    }
}
```

&nbsp;

Now that we can find the predecessor, we can finish the last part of the Remove method. Below is the code for replacing the node to be deleted with the predecessor. Note that we don't actually `delete` anything there like we did in the first two cases. Instead, we replace the `data` variable of the existing node with the predecessor's `data`. Since we now have two nodes with the same value, we call the remove method on the left subtree and remove the predecessor.

&nbsp;

```cpp
} else {
    Node<T>* pred = FindPredecessor(root->left);
    root->data = pred->data;
    root->left = Remove(root->left, pred->data);
}
```

&nbsp;

#### Emptying The Tree

Now it's time to finish the Empty method that was put in the destructor. To empty the tree, we need to recursively delete every item in the tree from heap memory due to the lack of garbage collection in C++. This is much easier than removing an individual node because we can start at the leaf nodes and work back up the tree using a postorder traversal. Postorder traversals visit the left node, the right node, then the root node. Finally, we'll set the root variable to `nullptr`.

Basic postorder traversal:
1. Recursive call on left subtree
2. Recursive call on right subtree
3. Visit node (in this case, delete the node)

&nbsp;

```cpp
template <typename T>
void BST<T>::Empty() {
    Empty(root);
    root = nullptr;
}

//Using postorder traveral to delete all objects on the heap
template <typename T>
void BST<T>::Empty(Node<T>* root) {
    if (root == nullptr) {
        return;
    } else {
        Empty(root->left);
        Empty(root->right);
        delete root;
    }
}
```

&nbsp;

#### Printing The Nodes

The final method for our templated tree is a Print method. In this implementation, I used an inorder traversal to print the nodes. As opposed to the postorder traversal, an inorder traversal goes left child, root node, right child. This will print the values in the tree from least to greatest. The difference between postorder and inorder only involves flipping two lines of code. For the inorder traversal, the 'visit' happens before the recursive call on the right subtree:

Basic inorder traversal:
1. Recursive call on left subtree
2. Visit node (in this case, print the node to console)
3. Recursive call on right subtree

&nbsp;

```cpp
//Public print method
template <typename T>
void BST<T>::Print() {
    if (root == nullptr) {
        std::cout << "Empty Tree" << std::endl;
        return;
    } else {
        Print(root);
    }
}

//Private helper method
template <typename T>
void BST<T>::Print(Node<T>* root) {
    if (root == nullptr) {
        return;
    } else {
        Print(root->left);
        std::cout << root->data << " ";
        Print(root->right);
    }
}
```

&nbsp;

#### Conclusion

This complete templated Binary Search Tree can be used with any data type in C++ that can be compared with comparison operators (==, >, <, etc). To use the templated BST with custom class objects, the comparison operators will need to be overloaded in that class to allow objects to be compared with each other. Operator overloading isn't covered in this post, but there are many good resources online covering this topic. 

Below is some example code showing the versatility of a templated class.

&nbsp;

```cpp
int main (int argc, char** argv) {
    
    cout << endl << endl;
    cout << "Integer Tree:";
    cout << endl << endl;
    //Create an integer BST with an inital root of 25
    BST<int>* tree = new BST<int>(25);
    tree->Insert(20);
    tree->Insert(12);
    tree->Insert(-10);
    tree->Insert(50);
    
    tree->Print();
    cout << endl << endl;
    
    tree->Remove(12);
    tree->Print();
    cout << endl << endl;
    
    Node<int>* node = tree->Find(2);
    if (node == nullptr) {
        cout << "not found" << endl;
    } else {
        cout << node->data << endl;
    }
    delete(tree);
    
    cout << endl << endl << endl << endl;
    cout << "Character Tree:";
    cout << endl << endl;
    
    //Create a character BST
    BST<char>* cTree = new BST<char>('k');
    cTree->Print();
    cout << endl << endl;
    
    cTree->Insert('a');
    cTree->Insert('z');
    cTree->Insert('d');
    cTree->Insert('s');
    
    cTree->Print();
    cout << endl << endl;
    
    cTree->Remove('k');
    
    cTree->Print();
    cout << endl << endl;
    
    delete(cTree);
}
```

&nbsp;

Thanks for reading!
