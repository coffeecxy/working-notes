二叉树的一些算法
===
二叉树是一种使用的相当广泛的数据结构，这里是它的一些算法的实现。

## 遍历算法
二叉树的遍历分为前序，中序，后序遍历。

* 前序，也叫做先根遍历，就是按照，根，左子树，右子树这样递归的遍历。
* 中序，也叫做中根遍历，就是按照，左子树，根，右子树这样递归的遍历。
* 后序，也叫做后根遍历，就是按照，左子树，右子树，根这样递归的遍历。

### 递归的实现
传统的，我们都会按照递归的方式来实现这些遍历算法。这写算法的实现都非常直接而且简单，所以就不详细的讨论了。

### 非递归实现

非递归实现二叉树的遍历需要使用到的就是栈这种LIFO的数据结构，不同的人有不同实现方式，在[这篇博客](http://www.cnblogs.com/dolphin0520/archive/2011/08/25/2153720.html)中，作者详细的讲解了他的实现过程。可以看出来对于后根遍历，实现起来比较复杂，而且要即兴的写出来也会比较麻烦。

经过思考，我也想出了一种非递归实现二叉树遍历的算法，我采用的方式不是像上面的博主那样完全的将实现变得不递归化。我的算法使用的是迭代的算法，但是很好的模拟了递归的算法，所以理解起来和写起来都十分直观。

首先，定义树的一个节点
```c++
struct Node {
	int value;
	Node* left;
	Node* right;

	int stage;
};
```
要注意的是，我定义的节点多了一个`stage`属性，我就是使用这个属性来辅助完成用迭代模拟递归的。

* stage=0，表示当前节点，其左子树，右子树都没有被访问过
* stage=1，表示当前节点被访问过了
* stage=2，表示其左子树被访问过了
* stage=0，表示其又子树被访问过了

#### 先根遍历

```c++
void preOrder(Node* root) {
	stack<Node*> sta;

	if (root != NULL) {
		sta.push(root);
	}

	while (!sta.empty()) {
		Node* topNode = sta.top();

		//是Null的时候不处理 
		if (topNode == NULL) {
			sta.pop();
			continue;
		}

		switch (topNode->stage) {
		case 0: //都没有访问 
			printf("%d\n", topNode->value); //访问自己 
			sta.pop();
			sta.push(topNode->right); //访问右子树 
			sta.push(topNode->left); //访问左边的子树 			
			//topNode->stage = 3;
			break;
		case 1: //自己访问完了 
			break;
		case 2: //左子树访问完了 	
			break;
		case 3: //右子树访问完了 
			break;
		default:
			break;
		}
	}
}
```

注意上面的代码,对于每个节点的左右儿子，都是要压入栈的，如果其是`NULL`，则直接不处理，并且弹出栈，这样做是为了避免判断左右儿子是不是`NULL`，让代码更美观。

当一个节点都没有被访问的时候，因为是先序遍历，所以先访问自己（`printf`语句），因为后面不用再使用它了，所以弹出，然后先访问左子树，再访问右子树，这里主要到栈的LIFO属性，所以先将右儿子压入，再将左儿子压入。这样左子树就会先处理了。

因为先序遍历很直接，所以注意到栈的LIFO属性之后，就可以直接写出来了，不需要使用stage都可以。

#### 中根遍历

```c++
void midOrder(Node* root) {

	stack<Node*> sta;

	if (root != NULL) {
		sta.push(root);
	}

	while (!sta.empty()) {
		Node* topNode = sta.top();

		//是Null的时候不处理 
		if (topNode == NULL) {
			sta.pop();
			continue;
		}

		switch (topNode->stage) {
		case 0: //都没有访问 
			sta.push(topNode->left); //访问左边的子树 
			topNode->stage = 2;
			break;
		case 1: //自己访问完了 	
			sta.pop();
			sta.push(topNode->right); //访问右子树 	
			topNode->stage = 3;
			break;
		case 2: //左子树访问完了 
			printf("%d\n", topNode->value); //访问自己 
			topNode->stage = 1;
			break;
		case 3: //右子树访问完了 
			continue;
			break;
		default:
			break;
		}
	}
}
```

在都没有访问的时候，要访问左子树。

左子树访问完了，那么就要访问自己。

自己访问完了，访问右子树。

上面的代码就是按照这个方式写出来的。需要注意的是在什么时候把自己从`stack`上面`pop()`出来，我们需要在访问完了自己，然后在要把`右儿子`压入之前把自己`pop()`出来。也就是在真的需要`pop`的时候才`pop`.如果在`case 2:`的时候就`pop`了，那么进入`case 1:`的时候`topNode`已经不是我们需要的node了。

#### 后根遍历
在博主提出的算法中，后根遍历实现起来很复杂，但是本文提供的算法，其实现起来很直观。
```c++
void postOrder(Node* root) {
	
	stack<Node*> sta;
	if (root != NULL) {
		sta.push(root);
	}

	while (!sta.empty()) {
		Node* topNode = sta.top();
		//是Null的时候不处理 
		if (topNode == NULL) {
			sta.pop();
			continue;
		}

		switch (topNode->stage) {
		case 0: //都没有访问 
			sta.push(topNode->right); //访问右子树 
			sta.push(topNode->left); //访问左边的子树 
			topNode->stage = 3;
			break;
		case 1: //自己访问完了 	
			break;
		case 2: //左子树访问完了 			
			break;
		case 3: //右子树访问完了 
			printf("%d\n", topNode->value); //访问自己 
			sta.pop();
			break;
		default:
			break;
		}
	}
}
```
在都没有访问的时候，访问左子树，在访问右子树。

访问完了右子树，访问自己。


#### 测试代码
```c++
int main() {
	Node node[8];
	for (int i = 1; i <= 7; i++) {
		node[i].value = i;
		node[i].left = node[i].right = NULL;
		node[i].stage = 0;
	}
	node[1].left = &node[2];
	node[1].right = &node[3];

	node[2].left = &node[4];
	node[2].right = &node[5];

	node[3].left = &node[6];
	node[3].right = &node[7];

	postOrder(&node[1]);

	return 0;
}
```

将上面的函数调用改成相应的就可以了。