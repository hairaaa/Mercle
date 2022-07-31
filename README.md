# Mercle Tree

在默克尔树实现时主要参考了csdn里的代码，我自己主要是搞清楚其每一步骤的功能与具体实现方法。

参考资料：

https://blog.csdn.net/qq_46140765/article/details/123828992?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165909493716782425172097%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=165909493716782425172097&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~pc_rank_34-2-123828992-null-null.142^v35^pc_rank_34&utm_term=Merkle%20Tree%E6%9E%84%E5%BB%BA%E5%AE%9E%E7%8E%B0&spm=1018.2226.3001.4187

结果：

![image](https://user-images.githubusercontent.com/104775629/182012132-76393dc5-f1c4-4242-95c7-f6716d5a997b.png)

具体实现步骤的描述：

由main函数开始逐步观察：

int main()

{

  //输入一些数据，数量长度不固定

	string check_str = "";
  
	cout << "输入 Merkle Tree的叶子结点的数据，以‘;’作为结束符: " << endl;
  
	vector<string> v;
  

	while (1) //输入叶子节点
  
	{
  
		string str;
    
		cin >> str;
    
		if (str != ";")
    
		{
    
			v.push_back(str);
      
		}
    
		else
    
		{
    
			break;
      
		}
    
	}
  
  //定义一个名为ntree的树
  
	tree ntree;
  
  //构造叶子节点
  
	ntree.buildBaseLeafes(v);
  
  '''
  
  void tree::buildBaseLeafes(vector<string> base_leafs) //建立叶子节点列表
  
{
  
	vector<node*> new_nodes;//节点
  

	cout << "叶子结点及对应的哈希值: " << endl;
  

	for (auto leaf : base_leafs) //给每一个字符串创建对应节点，并通过这个字符串设置哈希值
  
	{
  
		node* new_node = new node;
  
		new_node->setHash(leaf);//将这个节点的hash值计算出来
  
  '''
  
  void node::setHash(string hash_str)
  
{
  
	this->hash_str = sha2::hash256_hex_string(hash_str);//用sha256计算出hash值
  
}
  
  '''
  
		cout << leaf << ":" << new_node->getHash() << endl;//获得其hash值并输出
  
  '''
  
  string node::getHash()
{
  
	return hash_str;
  
}

  '''
  
		new_nodes.push_back(new_node);//放回添加
  
	}
  

	base.push_back(new_nodes);
  
	cout << endl;
  
}
  
  '''
  
	cout << "构建Merkle树过程:" << endl << endl;
  
	ntree.buildTree();//构建树
  
  '''
  
  void tree::buildTree() //建造merkle tree
  
{
  
	do
  
	{
  
		vector<node*> new_nodes;
  
		makeBinary(base.end()[-1]); //传入尾元素 即一个节点列表

		for (int i = 0; i < base.end()[-1].size(); i += 2)
                                              
		{
                                              
			node* new_parent = new node; //设置父亲节点 传入最后一个元素 即一个节点列表的第i和i+1个
                                              
			base.end()[-1][i]->setParent(new_parent);
  
			base.end()[-1][i + 1]->setParent(new_parent);
  

			//通过两个孩子节点的哈希值设置父节点哈希值
  
			new_parent->setHash(base.end()[-1][i]->getHash() + base.end()[-1][i + 1]->getHash());
  
			//将该父节点的左右孩子节点设置为这两个
  
			new_parent->setChildren(base.end()[-1][i], base.end()[-1][i + 1]);
  
			//将new_parent压入new_nodes
  
			new_nodes.push_back(new_parent);

			cout << "将 " << base.end()[-1][i]->getHash() << " 和 " << base.end()[-1][i + 1]->getHash() << " 连接,得到对应父节点的哈希值 " << endl;
  
		}

		cout << endl;
  
		cout << "得到的对应父节点的哈希值:" << endl;
  
		printTreeLevel(new_nodes);

		base.push_back(new_nodes); //将新一轮的父节点new_nodes压入base

		cout << "该层的结点有 " << base.end()[-1].size() << " 个:" << endl;
  
	} while (base.end()[-1].size() > 1); //这样每一轮得到新一层的父节点，知道得到根节点 退出循环

	merkleRoot = base.end()[-1][0]->getHash(); //根节点的哈希值

	cout << "Merkle Root : " << merkleRoot << endl << endl;
  
}
  
  '''

	cout << endl;
  
	cout << "想验证的数据: " << endl;
  
	cin >> check_str; //输入想验证的叶子节点
  
	check_str = sha2::hash256_hex_string(check_str);

	cout << "想验证的数据的哈希值: " << check_str << endl;

	if (ntree.verify(check_str))//验证有无这个节点 树有无改变
  '''
  int tree::verify(string hash)
  
{
  
	node* el_node = nullptr;
  
	string act_hash = hash;

	for (int i = 0; i < base[0].size(); i++)
                                     
	{
                                     
		if (base[0][i]->getHash() == hash)
  
		{
  
			el_node = base[0][i];
  
		}
  
	}
  
	if (el_node == nullptr)
  
	{
  
		return 0;
  
	}

	cout << "使用到的哈希值:" << endl;
  
	cout << act_hash << endl;

	do  //验证merkle tree是否改变过 
  
	{
  
		//父节点的哈希是左孩子的哈希string+右孩子的哈希string
  
		//如果el_node的父节点的左节点是el_node
  
		if (el_node->checkDir() == 0)
  
		{
  
			//是左孩子就 做孩子的哈希string+右孩子的哈希string
  
			act_hash = sha2::hash256_hex_string(act_hash + el_node->getSibling()->getHash());
  
		}
  
		else
  
		{
  
			act_hash = sha2::hash256_hex_string(el_node->getSibling()->getHash() + act_hash);
  
		}

		std::cout << act_hash << endl;

		el_node = el_node->getParent();
  
	} while ((el_node->getParent()) != NULL); //到达根节点

	return act_hash == merkleRoot ? 1 : 0;
  
}
  '''
	{
  
		cout << endl << endl;
  
		cout << "Merkle树上存在验证的数据的叶子结点" << endl;
  
	}
  
	else
  
	{
  
		cout << "Merkle树上不存在验证的数据" << endl;
  
	}
  
	return 0;
  
}
