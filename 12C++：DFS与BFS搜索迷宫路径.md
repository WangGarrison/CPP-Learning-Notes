# 题目背景

```cpp
请输入迷宫的行列数(例如:5 5)
请输入迷宫的路径信息(0表示可以走，1表示不能走):
0 0 0 1 1
1 0 0 0 1
1 1 0 1 1
1 1 0 0 1
1 1 1 0 0
迷宫路径搜索中...
>>>如果没有路径，直接输出<<<
不存在一条迷宫路径！
>>>如果有路径，直接输出<<<
* * * 1 1
1 0 * 0 1
1 1 * 1 1
1 1 * * 1
1 1 1 * *
```

# 深度优先遍历进行搜索DFS

**思路：**

- 定义结点，结点属性包含坐标（x，y），值，以及状态（各个方向是否可走）；定义迷宫棋盘

- 规定先从右方向开始找，==右方向可以走就一直往右方向走（深度优先）==，右方向不行再是下，再是左，再是上，方向都不行就回退，用一个栈存储走过的结点

- 将第一个结点入栈，当栈不为空：取栈顶元素

- 如果栈顶元素右方向可以走，就将该结点的右方向设置为不可走（因为现在正在走过该方向），将该结点的右节点的左方向设置为不可走（因为从左边来的），然后将该结点的右方结点入栈（表示路径上有该结点），continue

- 如果走到死胡同，就出栈一个元素（回溯到上一个结点），探索上一个结点的其他方向

- 如果走到了row-1与col-1，表示走到了右下角的出口，结束

  ![image-20210316201452550](img/12C++%EF%BC%9A%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86%E6%90%9C%E7%B4%A2%E8%BF%B7%E5%AE%AB%E8%B7%AF%E5%BE%84.img/image-20210316201452550.png)

```cpp
#include <iostream>
#include <stack>
using namespace std;

//定义迷宫每一个节点的四个方向
const int RIGHT = 0;
const int DOWN = 1;
const int LEFT = 2;
const int UP = 3;

//迷宫每一个节点方向的数量
const int WAY_NUM = 4;

//定义节点行走状态
const int YES = 4;
const int NO = 5;

//迷宫
class Maze
{
private:
	//定义迷宫节点路径信息
	struct Node
	{
		int _x;   
		int _y;
		int _val; //节点的值, 0表示可以走，1表示不可以走
		int _state[WAY_NUM]; //记录节点四个方向的状态
	};

	Node **_pMaze; //动态生成迷宫路径，根据用户输入的行列数，动态产生迷宫
	int _row;
	int _col;
	stack<Node> _stack; //栈结构，辅助深度搜索迷宫路径
    
public:
	//初始化迷宫，根据用户输入的行列数，生成存储迷宫路径信息的二维数组
	Maze(int row, int col) :_row(row), _col(col)
	{
		_pMaze = new Node*[_row];
		for (int i = 0; i < _row; ++i)
		{
			_pMaze[i] = new Node[_col];
		}
	}
    
    //初始化迷宫路径节点信息
	void initNode(int x, int y, int val)
	{
		_pMaze[x][y]._x = x;
		_pMaze[x][y]._y = y;
		_pMaze[x][y]._val = val;
		//节点四个方向默认的初始化，默认为不能走
		for (int i = 0; i < WAY_NUM; ++i)
		{
			_pMaze[x][y]._state[i] = NO;
		}
	}
    
    //初始化迷宫值为0的节点四个方向的行走状态信息，值为1的结点表示墙，没必要设置状态
	void setNodeState()
	{
		for (int i = 0; i < _row; ++i)
		{
			for (int j = 0; j < _col; ++j)
			{
				if (_pMaze[i][j]._val == 1)//结点值为1，说明走不到该节点，没必要设置其state
				{
					continue;
				}

				if (j < _col - 1 && _pMaze[i][j + 1]._val == 0)//右方向可以走，注意j+1不要越界
				{
					_pMaze[i][j]._state[RIGHT] = YES;
				}

				if (i < _row - 1 && _pMaze[i + 1][j]._val == 0)//下方向可以走
				{
					_pMaze[i][j]._state[DOWN] = YES;
				}

				if (j > 0 && _pMaze[i][j - 1]._val == 0)//左方向可以走
				{
					_pMaze[i][j]._state[LEFT] = YES;
				}

				if (i > 0 && _pMaze[i - 1][j]._val == 0)//上方向可以走
				{
					_pMaze[i][j]._state[UP] = YES;
				}
			}
		}
	}
    
    // 深度搜索迷宫路径
	void searchMazePath()
	{
		if (_pMaze[0][0]._val == 1)//入口是墙，入口都不可走，直接返回
		{
			return;
		}
		_stack.push(_pMaze[0][0]);

		while (!_stack.empty())
		{
			Node top = _stack.top();
			int x = top._x;
			int y = top._y;

			//已经找到右下角出口的迷宫路径
			if (x == _row - 1 && y == _col - 1)
			{
				return;
			}

			//往右方向寻找
			if (_pMaze[x][y]._state[RIGHT] == YES)
			{
				_pMaze[x][y]._state[RIGHT] = NO;
				_pMaze[x][y + 1]._state[LEFT] = NO;
				_stack.push(_pMaze[x][y + 1]);
				continue;
			}

			//往下方向寻找
			if (_pMaze[x][y]._state[DOWN] == YES)
			{
				_pMaze[x][y]._state[DOWN] = NO;
				_pMaze[x + 1][y]._state[UP] = NO;
				_stack.push(_pMaze[x + 1][y]);
				continue;
			}

			//往左方向寻找
			if (_pMaze[x][y]._state[LEFT] == YES)
			{
				_pMaze[x][y]._state[LEFT] = NO;
				_pMaze[x][y - 1]._state[RIGHT] = NO;
				_stack.push(_pMaze[x][y - 1]);
				continue;
			}

			//往上方向寻找
			if (_pMaze[x][y]._state[UP] == YES)
			{
				_pMaze[x][y]._state[UP] = NO;
				_pMaze[x - 1][y]._state[DOWN] = NO;
				_stack.push(_pMaze[x - 1][y]);
				continue;
			}

			_stack.pop();
		}
	}

	//打印迷宫路径搜索结果
	void showMazePath()
	{
		if (_stack.empty())
		{
			cout << "不存在一条迷宫路径！" << endl;
		}
		else
		{
			while (!_stack.empty())
			{
				Node top = _stack.top();
				_pMaze[top._x][top._y]._val = '*';
				_stack.pop();
			}

			for (int i = 0; i < _row; ++i)
			{
				for (int j = 0; j < _col; ++j)
				{
					if (_pMaze[i][j]._val == '*')
					{
						cout << "* ";
					}
					else
					{
						cout << _pMaze[i][j]._val << " ";
					}
				}
				cout << endl;
			}
		}
	}
};

int main()
{
	cout << "请输入迷宫的行列数(例如：10 10):";
	int row, col, data;
	cin >> row >> col;

	Maze maze(row, col); // 创建迷宫对象

	cout << "请输入迷宫的路径信息(0表示可以走，1表示不能走):" << endl;
	for (int i = 0; i < row; ++i)
	{
		for (int j = 0; j < col; ++j)
		{
			cin >> data;
			// 可以初始化迷宫节点的基本信息
			maze.initNode(i, j, data);
		}
	}

	// 开始设置所有节点的四个方向的状态
	maze.setNodeState();

	// 开始从左上角搜索迷宫的路径信息了
	maze.searchMazePath();

	// 打印迷宫路径搜索的结果
	maze.showMazePath();

	return 0;
}
```

**运行结果：**

 <img src="img/12C++%EF%BC%9A%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86%E6%90%9C%E7%B4%A2%E8%BF%B7%E5%AE%AB%E8%B7%AF%E5%BE%84.img/image-20210316210620509.png" alt="image-20210316210620509" style="zoom:50%;" />

**深度优先遍历的缺点：**

如果是下面这样的迷宫，那么输出的路径显然不是最优的路径

 <img src="img/12C++%EF%BC%9A%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86%E6%90%9C%E7%B4%A2%E8%BF%B7%E5%AE%AB%E8%B7%AF%E5%BE%84.img/image-20210316211753143.png" alt="image-20210316211753143" style="zoom: 40%;" />

# 广度优先遍历进行搜索BFS

深度优先遍历：一般依赖栈或者递归

广度优先遍历：层层扩张的方式，依赖队列结构

**思路：**

- 第一个结点入队，==该结点可以走的方向的结点都要入队，把队列中的结点挨个进行探查方向==，处理完一个结点出队一个结点

- 要用一个数组记录路径（通过记录该结点是从哪个节点走过来的（arr[x*col+y] = 上一个结点的坐标）来记录路径）

  ![image-20210316220101987](img/12C++%EF%BC%9A%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86%E6%90%9C%E7%B4%A2%E8%BF%B7%E5%AE%AB%E8%B7%AF%E5%BE%84.img/image-20210316220101987.png)

```cpp
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

// 定义方向
const int RIGHT = 0;
const int DOWN = 1;
const int LEFT = 2;
const int UP = 3;
const int WAY_NUM = 4;

// 定义行走状态
const int YES = 4;
const int NO = 5;

// 迷宫
class Maze
{
public:
	Maze(int row, int col):_row(row), _col(col)
	{
		_pMaze = new Node*[_row];
		for (int i = 0; i < _row; ++i)
		{
			_pMaze[i] = new Node[_col];
		}

		// node._x*_col + node._y
		_pPath.resize(_row * _col);  //开辟空间
	}

	void initNode(int x, int y, int val)
	{
		_pMaze[x][y]._x = x;
		_pMaze[x][y]._y = y;
		_pMaze[x][y]._val = val;
		for (int i = 0; i < WAY_NUM; ++i)
		{
			_pMaze[x][y]._state[i] = NO;
		}
	}

	void setNodeState()
	{
		for (int i = 0; i < _row; ++i)
		{
			for (int j = 0; j < _col; ++j)
			{
				if (_pMaze[i][j]._val == 1)
				{
					continue;
				}

				if (j < _col - 1 && _pMaze[i][j + 1]._val == 0)
				{
					_pMaze[i][j]._state[RIGHT] = YES;
				}

				if (i < _row - 1 && _pMaze[i + 1][j]._val == 0)
				{
					_pMaze[i][j]._state[DOWN] = YES;
				}

				if (j > 0 && _pMaze[i][j - 1]._val == 0)
				{
					_pMaze[i][j]._state[LEFT] = YES;
				}

				if (i > 0 && _pMaze[i - 1][j]._val == 0)
				{
					_pMaze[i][j]._state[UP] = YES;
				}
			}
		}
	}

    //广度优先遍历搜索路径
	void searchMazePath()
	{
		if (_pMaze[0][0]._val == 1)
		{
			return;
		}
		_queue.push(_pMaze[0][0]);

		while (!_queue.empty())
		{
			Node front = _queue.front();
			int x = front._x;
			int y = front._y;

			// 右方向
			if (_pMaze[x][y]._state[RIGHT] == YES)
			{
				_pMaze[x][y]._state[RIGHT] = NO;
				_pMaze[x][y + 1]._state[LEFT] = NO;
				// 记录下一个结点是从哪个节点走过来的
				_pPath[x*_col + y + 1] = _pMaze[x][y];
				_queue.push(_pMaze[x][y + 1]);
				if (check(_pMaze[x][y + 1]))
					return;
			}

			// 下方向
			if (_pMaze[x][y]._state[DOWN] == YES)
			{
				_pMaze[x][y]._state[DOWN] = NO;
				_pMaze[x + 1][y]._state[UP] = NO;
				_pPath[(x + 1)*_col + y] = _pMaze[x][y];
				_queue.push(_pMaze[x + 1][y]);
				if (check(_pMaze[x + 1][y]))
					return;
			}

			// 左方向
			if (_pMaze[x][y]._state[LEFT] == YES)
			{
				_pMaze[x][y]._state[LEFT] = NO;
				_pMaze[x][y - 1]._state[RIGHT] = NO;
				_pPath[x*_col + y - 1] = _pMaze[x][y];
				_queue.push(_pMaze[x][y - 1]);
				if (check(_pMaze[x][y - 1]))
					return;
			}

			// 上方向
			if (_pMaze[x][y]._state[UP] == YES)
			{
				_pMaze[x][y]._state[UP] = NO;
				_pMaze[x - 1][y]._state[DOWN] = NO;
				_pPath[(x - 1)*_col + y] = _pMaze[x][y];
				_queue.push(_pMaze[x - 1][y]);
				if (check(_pMaze[x - 1][y]))
					return;
			}

			// 出队列
			_queue.pop();
		}
	}

	void showMazePath()
	{
		if (_queue.empty())
		{
			cout << "不存在一条迷宫路径！" << endl;
		}
		else
		{
			// 回溯寻找迷宫路径节点
			int x = _row - 1;
			int y = _col - 1;  //x，y现在处于右下角出口
			for (;;)
			{
				_pMaze[x][y]._val = '*';
				if (x == 0 && y == 0)
					break;
				Node node = _pPath[x*_col + y];  //取出_pPath存的路径的上一结点
				x = node._x;
				y = node._y;
			}

			for (int i = 0; i < _row; ++i)
			{
				for (int j = 0; j < _col; ++j)
				{
					if (_pMaze[i][j]._val == '*')
					{
						cout << "* ";
					}
					else
					{
						cout << _pMaze[i][j]._val << " ";
					}
				}
				cout << endl;
			}
		}
	}
private:
	// 定义迷宫节点路径信息
	struct Node
	{
		int _x;
		int _y;
		int _val; // 节点的值
		int _state[WAY_NUM]; // 记录节点四个方向的状态
	};

	// 检查是否是右下角的迷宫出口节点
	bool check(Node &node)
	{
		return node._x == _row - 1 && node._y == _col - 1;
	}

	Node **_pMaze;
	int _row;
	int _col;
	queue<Node> _queue; // 广度遍历依赖的队列结构
	vector<Node> _pPath; // 记录广度优先遍历时，节点的行走信息
};

int main()
{
	cout << "请输入迷宫的行列数(例如：10 10):";
	int row, col, data;
	cin >> row >> col;

	Maze maze(row, col); // 创建迷宫对象

	cout << "请输入迷宫的路径信息(0表示可以走，1表示不能走):" << endl;
	for (int i = 0; i < row; ++i)
	{
		for (int j = 0; j < col; ++j)
		{
			cin >> data;
			// 可以初始化迷宫节点的基本信息
			maze.initNode(i, j, data);
		}
	}

	// 开始设置所有节点的四个方向的状态
	maze.setNodeState();

	// 开始从左上角搜索迷宫的路径信息了
	maze.searchMazePath();

	// 打印迷宫路径搜索的结果
	maze.showMazePath();

	return 0;
}
```

**运行结果：**

 <img src="img/12C++%EF%BC%9A%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E9%81%8D%E5%8E%86%E6%90%9C%E7%B4%A2%E8%BF%B7%E5%AE%AB%E8%B7%AF%E5%BE%84.img/image-20210316225857795.png" alt="image-20210316225857795" style="zoom:50%;" />