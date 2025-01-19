---
title: 校园导航系统
shortTitle: 6.校园导航系统
date: 2025-01-17
category:
  - 课程设计
tag:
  - 课程设计
---

## 校园导航系统
采用C++语言涉及了数据库相关的操作，多使用面向对象语法。
使用最短路径等算法实现。
## 演示示例
![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182221047.png)

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182221844.png)

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182221855.png)
## 代码示例

### 1、弧结点和顶点节点
ArcNode.h

```cpp
#pragma once
class ArcNode
{
private:
	int weight;
	int mark;					//访问标记   0 代表未访问
	int ivex, jvex;				//该边依附两个顶点的的位置
	ArcNode* ilink, * jlink;	//分别指向依附这2个顶点的下一条边
public:
	ArcNode() :ivex(0), jvex(0), ilink(nullptr), jlink(nullptr), mark(0), weight(0) {}
	void set_weight(int w) {
		this->weight = w;
	}
	int get_weight() {
		return weight;
	}
	void set_mark(int i) {
		this->mark = i;
	}
	int get_mark() {
		return mark;
	}
	void set_ivex(int i1) {
		this->ivex = i1;
	}
	int get_ivex() {
		return ivex;
	}
	void set_jvex(int i2) {
		this->jvex = i2;
	}
	int get_jvex() {
		return jvex;
	}
	void set_ilink(ArcNode* it) {
		this->ilink = it;
	}
	ArcNode* get_ilink() {
		return ilink;
	}
	void set_jlink(ArcNode* ik) {
		this->jlink = ik;
	}
	ArcNode* get_jlink() {
		return jlink;
	}
};
```

VertexNode.h

```cpp
#pragma once
#include"ArcNode.h"
#include<string>
using std::string;
class VertexNode
{
private:
	int id;
	string data;
	string content;
public:
	VertexNode():id(0),data(""),content("") {}
	void set_data(string ata) {
		this->data = ata;
	}
	string get_data() {
		return data;
	}
	void set_content(string ta) {
		this->content = ta;
	}
	string get_content() {
		return content;
	}
	void set_id(int ik) {
		this->id = ik;
	}
	int get_id() {
		return id;
	}
};
```

### 2、Map节点

Map.h

```cpp
#pragma once
#include"ArcNode.h"//边类
#include"VertexNode.h"//顶点类
#include"Databases.h"//数据库类
#include"User.h"
class Map
{
private:
	Databases database;//定义数据库的对象，连接数据库
public:
	int ArcArrary[99][99] = { 0 };//邻接矩阵
	VertexNode* arrary;//地图的顶点数组
	int vernum;//地点个数
	int arcnum;//边个数

	bool visited[999] = { false };//标记每个顶点是否被访问过
	int dist[999] = { 0 };//存储源点到各点的最短距离
	int path[999] = { 0 };//存储最短路径上当前节点的前一个节点

	int pathaway[99][99] = { 0 };//最短路径上的前驱节点
	int dist_f[99][99] = { 0 };//两个顶点之间的最短距离
public:
	void Read();//读取数据函数
	void Clear();//销毁函数
	int GetDataIndex(string data);//找到此建筑的下标
	bool RepatGetData(string data);//查重
	void Display_schoolmap();//打印平面地图
	void Search_route();//显示所有地点的编号以及地点间的路径距离
	void Dijkstra(int v);//迪杰斯特拉算法,找最短路径
	void Floyed();//弗洛伊德算法
	void Print_Floyed(int start, int end);//调用佛洛依德
	bool Print_Choice_place(int start, int end, int& sum, string& str);//打印必经点的路径
	void DFS(int i, int j, int path[], int& len, int& sum, bool& f);//深度优先遍历
	void Print_DFS(int i, int j);//打印两点间所有可能存在的路径
	void BFS(int start, int limit);//广度优先遍历
public://管理员操作
	void Add_newVertex();//添加新的顶点
	void Add_newArc();//添加新边
	void Delete_oldVertex(string data);//删除旧的顶点
	bool Delete_oldArc(int i, int j);//删除旧的边
	bool Alter_ver(int i);//修改顶点
	bool Alter_arc(int i, int j);//修改边
	void Writing_information(int i);//描绘信息
public://登录功能
	void registered();//注册
	bool login(string& aCC);//登录
	void modify(string acc);//修改
	void logout(string acc);//注销
};
```
Map.cpp

```cpp
#define _CRT_SECURE_NO_WARNINGS
#define MAXINFNUM 99999
#include"VertexNode.h"
#include"Map.h"
#include<iostream>
#include<iomanip>//setw对齐
#include<queue>
#include<vector>
#include<cstring>
#include<stack>
#include<string>
#include <random>//C++更加标准的随机数生成
#include<ctime>//获取当地时间
#include<conio.h>
#include<Windows.h>
using namespace std;
const int MAXNUM = 999;
void Map::Read()
{
	vernum = 0;
	arcnum = 0;
	arrary = new VertexNode[MAXNUM];
	for (int i = 0; i < 99; i++) {
		for (int j = 0; j < 99; j++) {
			ArcArrary[i][j] = MAXINFNUM;
		}
	}
	database.Read_databases_place(arrary, vernum);
	database.Read_databases_pathway(arcnum,vernum,arrary,ArcArrary);
}

void Map::Clear()
{
	vernum = 0;
	arcnum = 0;
	for (int i = 0; i < 99; i++) {
		for (int j = 0; j < 99; j++) {
			ArcArrary[i][j] = MAXINFNUM;
		}
	}
}

int Map::GetDataIndex(string data)
{
	//根据名字查找
	for (int i = 0; i < vernum; i++) {
		if (arrary[i].get_data() == data) {
			return i;
		}
	}
	bool isDigit = true;
	bool isHan = true;
	//判断输入是否是汉字
	for (char ch : data) {
		if (ch >= 0x4e00 && ch <= 0x9fff) {
			isHan = false;
			break;
		}
	}
	if (isHan) {
		//判断输入是否全部是数字
		for (char c : data) {
			if (!isdigit(c)) {
				isDigit = false;
				break;
			}
		}
		//如果输入全部是数字，则将其转化为整型
		int num = 0;
		if (isDigit) {
			num = stoi(data);
			//根据id查找
			for (int i = 0; i < vernum; i++) {
				if (arrary[i].get_id() == stoi(data)) {
					return i;
				}
			}
		}
	}
	return -1;
}

bool Map::RepatGetData(string data)
{
	for (int i = 0; i < vernum; i++) {
		if (data == arrary[i].get_data()) {
			return true;
		}
	}
	return false;
}

bool check(string str) {//检查权值是否为正整数
	int i;
	bool f = true;
	for (i = 0; i < str.size(); i++) {
		//如果不是数字
		if (str[i] >= '0' && str[i] <= '9' && str[0] != '0') {
			f = true;
		}
		else {
			f = false;
			break;
		}
	}
	return f;
}

void school_map() {
	cout << "————————————南阳理工学院（西北校区）平面图—————————-"<< endl;
	cout << "              《北门》                                                   |" << endl;
	cout << "                 |                   |------《4教》                      |"<< endl;
	cout << "                 |--------------------                                   |"<< endl;
	cout << "                 |              《生化学院》                             |"<< endl;
	cout << " --------《1教》--                   / |                                 |"<< endl;
	cout << " |               |                  /  |                                 |"<< endl;
	cout << "《3教》          |                 /   |                                 |"<< endl;
	cout << " |               |                /    |                                 |"<< endl;
	cout << " |               ----------------/     |                                 |"<< endl;
	cout << " |               |                     |                                 |"<< endl;
	cout << " |           《中关村》                |                                 |"<< endl;
	cout << " |                |                    |                                 |"<< endl;
	cout << "  \\               |                《西北操场》                          |"<< endl;
	cout << "   \\              \\                  |                                   |"<< endl;
	cout << "     \\             |                 /                                   |"<< endl;
	cout << "       ---------《小礼堂》------------                                   |"<< endl;
	cout << "       |             |                                                   |"<< endl;
	cout << "      /               \\                                                  |"<< endl;
	cout << "     /                 \\                                                 |"<< endl;
	cout << "《西北餐厅》            \\                                                |"<< endl;
	cout << "     |     \\             \\                                               |"<< endl;
	cout << "     |   《下沉广场》---《校医务室》                                     |"<< endl;
	cout << "     |        /             |                                            |"<< endl;
	cout << "《女生公寓》-               |                                            |"<< endl;
	cout << "     \\                      |                                            |"<< endl;
	cout << "      |                     |                                            |"<< endl;
	cout << " 《土木工程学院》           |                                            |"<< endl;
	cout << "      | \\                   |                                            | "<< endl;
	cout << "      |   \\                 /                                            |"<< endl;
	cout << "      |     \\--《画室》----                                              |"<< endl;
	cout << "      |           |                                                      |"<< endl;
	cout << "      |      《建筑学院》                                                |"<< endl;
	cout << "      |          /                                                       |"<< endl;
	cout << " 《智能制造学院》                                                        |"<< endl;
	cout << "-------------------------------------------------------------------------" << endl;
	cout << endl;
	cout << "————————————南阳理工学院（东南校区）平面图—————————-—" << endl;
	cout << "|齐贤广场*-----                                                            |" << endl;
	cout << "|              \\                                                           |" << endl;
	cout << "|               *信息工程学院-*计算机与软件学院---*菜鸟驿站       *教师公寓|" << endl;
	cout << "|               |             |                                   |        |" << endl;
	cout << "|               *数理学院-----|                                   |        |" << endl;
	cout << "|               |             |                                   |        |" << endl;
	cout << "|                --           |                                   |        |" << endl;
	cout << "|                  |          |                                   |        |" << endl;
	cout << "|张仲景国医国药学院*----------|-----------------------------------*学生公寓|" << endl;
	cout << "|                  | |   梦  ||               |                            |" << endl;
	cout << "|              ----\\ |       ||               |                            |" << endl;
	cout << "|             /   --*|   溪  |*               *东南操场                    |" << endl;
	cout << "|            /      ||       ||               \\                            |" << endl;
	cout << "|           /       ||   湖                   \\                            |" << endl;
	cout << "|          /       /  -------------------------------*东南餐厅             |" << endl;
	cout << "|         /       /                                  |                     |" << endl;
	cout << "|汇森楼*------   /        ---------------------------|                     |" << endl;
	cout << "|             \\ /         |                                                |" << endl;
	cout << "|              *图书馆 ---*体育馆                                          |" << endl;
	cout << "---------------------------------------------------------------------------" << endl;
}

void Map::Display_schoolmap()//要修改为学校平面地图
{
	system("cls");
	school_map();
	string cs = "";
	bool f = false;
	cout << "是否查看地点介绍(y/n):";
	while (1) {
		cin >> cs;
		if (cs == "y") {
			cout << "请输入你要查看的地点名字：";
			string name = "";
			cin >> name;
			for (int j = 0; j < vernum; j++) {
				if (arrary[j].get_data() == name) {
					cout << "地点：" << name << endl;
					cout << "简介：" << arrary[j].get_content() << endl;
					f = true;
					break;
				}
			}
		}
		else {
			f = false;
			cout << "退出此界面。" << endl;
			Sleep(400);
			break;
		}
		if (f) {
			cout << "是否继续查看地点(y/n):";
		}
	}
}

void Map::Search_route()
{
	int fake_arr[99][99] = { 0 };
	for (int i = 0; i < vernum; i++) {
		for (int j = 0; j < vernum; j++) {
			fake_arr[i][j] = ArcArrary[i][j];
		}
	}
	int lj = 1;
	cout << "校园地图现有的建筑物" << vernum << "个,路径数 " << arcnum << "个" << endl;
	cout << "+-------------------------------------------------------------+" << endl;
	cout << left <<setw(10)<<"| 编号 |         建筑物    <---->    建筑物          |  距离  |" << endl;
	cout << "+-------------------------------------------------------------+" << endl;
	for (int i = 0; i < vernum; i++) {
		for (int j = 0; j < vernum; j++) {
			if (fake_arr[i][j] != MAXINFNUM) {
				cout << left <<"| " <<setw(4)<<lj++ << " | " << setw(20) << arrary[i].get_data() << " | " <<setw(20)<< arrary[j].get_data() << " |   " <<setw(3)<< ArcArrary[i][j] << "  |" << endl;
				cout << "+-------------------------------------------------------------+" << endl;
				fake_arr[j][i] = fake_arr[i][j] = MAXINFNUM;
			}
		}
	}
	system("pause");
}

void Map::Dijkstra(int v)//求某个顶点到其他顶点的最短路径,并将所有的最短路径全部打印
{
	for (int i = 0; i < vernum; i++) {//初始化3个辅助数组
		visited[i] = false;
		dist[i] = MAXINFNUM;
		path[i] = -1;
	}
	dist[v] = 0;//源点到自身为0
	for (int i = 0; i < vernum; i++) {//访问源点的第一条边
		if (i == v) continue;
		if (ArcArrary[v][i] != MAXINFNUM) {
			dist[i] = ArcArrary[v][i];
			path[i] = v;
		}
	}
	//将源点加入已访问集合
	for (int i = 1; i < vernum; i++) {
		int minDist = MAXINFNUM;
		int u = v;
		for (int j = 0; j < vernum; j++) {//找到未访问集合中距离源点最近的节点
			if (!visited[j] && dist[j] < minDist) {
				minDist = dist[j];//找到最小的权值
				u = j;//记录最小权值的父节点
			}
		}
		visited[u] = true;//将该节点加入已访问集合
		for (int w = 0; w < vernum; w++) {//更新剩余未访问集合中节点的最短距离和前一个节点
			//因为u已经更新了父节点为j，所以要看uw的路径是否存在
			if (!visited[w] && ArcArrary[u][w] != MAXINFNUM && dist[u] + ArcArrary[u][w] < dist[w]) {
				dist[w] = dist[u] + ArcArrary[u][w];//每个能到达的顶点的下标为w，dist[w]为起点v到终点w的权值和
				path[w] = u;//path记录了每个顶点距离最近的父节点的下标
			}
		}
	}
	//输出源点到各个顶点的最短路径
	cout << "起点：<" << arrary[v].get_data() << ">" << endl;
	cout << "终点：" << endl;
	for (int i = 0; i < vernum; i++) {
		if (i != v) {
			cout << i+1 << " :  " << arrary[i].get_data() << endl;
			if (dist[i] == MAXINFNUM) {
				cout << "No Path." << endl;
			}
			else {
				cout << arrary[v].get_data();
				stack<int> p; // 使用栈来存储路径
				int k = i;
				while (path[k] != -1) {
					p.push(k);
					k = path[k];
				}
				while (!p.empty()) {
					cout << right << "->" << arrary[p.top()].get_data();
					p.pop();
				}
				cout << "（" << dist[i] << "）" << endl;
			}
		}
	}
	cout << endl;
}

void Map::Floyed()
{
	for (int i = 0; i < vernum; ++i) {
		//初始化
		for (int j = 0; j < vernum; ++j) {
			dist_f[i][j] = MAXINFNUM;
			pathaway[i][j] = -1;
		}
		for (int k = 0; k < vernum; k++) {
			if (ArcArrary[i][k] != MAXINFNUM && dist_f[i][k] > ArcArrary[i][k]) {
				dist_f[i][k] = dist_f[k][i] = ArcArrary[i][k];
				pathaway[i][k] = k;
			}
		}
		dist_f[i][i] = 0;
	}
	//k为中转点，i为起点，j为终点，循环比较dist_f[i][j]和dist_f[i][k] + dist_f[k][j]，
	//如果dist_f[i][k] + dist_f[k][j]为更小值，则把dist_f[i][k] + dist_f[k][j]覆盖保存在dist_f[i][j]中。
	for (int k = 0; k < vernum; ++k) {
		for (int i = 0; i < vernum; ++i) {
			for (int j = 0; j < vernum; ++j) {
				if (dist_f[i][j] > dist_f[i][k] + dist_f[k][j]) {
					dist_f[i][j] = dist_f[i][k] + dist_f[k][j];
					pathaway[i][j] = pathaway[i][k];
				}
			}
		}
	}
}

void Map::Print_Floyed(int start, int end)
{
	Floyed();//调用弗洛伊德算法，求最短路径
	if (dist_f[start][end] == MAXINFNUM) {
		cout << "没有此路径。" << endl;
	}
	else {
		cout << "\n最优出行路线 : " << arrary[start].get_data() << "->" << arrary[end].get_data() << endl;
		cout << arrary[start].get_data() << "->";
		int cur = start;
		while (pathaway[cur][end] != end) {
			cur = pathaway[cur][end];
			//上次报错信息是因为未判断cur是否为-1了
			cout << arrary[cur].get_data() << "->";
		}
		cout << arrary[end].get_data() << " （" << dist_f[start][end] << "）" << endl;
	}
}

bool Map::Print_Choice_place(int start, int end, int& sum, string& str)//必经点路径
{
	Floyed();//调用弗洛伊德算法，求最短路径
	if (dist_f[start][end] == MAXINFNUM) {
		return false;
	}
	else {
		cout << arrary[start].get_data() << "->";
		int cur = start;
		while (pathaway[cur][end] != end) {
			cur = pathaway[cur][end];
			//上次报错信息是因为未判断cur是否为-1了
			cout << arrary[cur].get_data() << "->";
			str += arrary[cur].get_data() + "->";
		}
		cout << arrary[end].get_data() << "  (" << dist_f[start][end] << ")" << endl;
		str += arrary[end].get_data() + "->";
		sum += dist_f[start][end];//就算此路径
		return true;
	}
}

void Map::DFS(int i, int j, int path[], int& len, int& sum, bool& f)
{
	//深度优先搜索，先读取i到j的最深路径，然后回溯
	visited[i] = true;
	path[len++] = i;//前驱数组，存放前一个顶点的下标
	if (i == j) {//递归实现找到全部的路径
		f = true;
		for (int k = 0; k < len - 1; k++) {
			cout << arrary[path[k]].get_data() << " -> ";
		}
		cout << arrary[path[len - 1]].get_data() << "  (" << sum << ")" << endl;
	}
	for (int k = 0; k < vernum; k++) {
		if (ArcArrary[i][k] != MAXINFNUM && !visited[k]) {
			sum += ArcArrary[i][k];//向深处遍历则增加权值
			DFS(k, j, path, len, sum, f);
			sum -= ArcArrary[i][k];//回溯就要减去加上的权值
		}
	}
	len--;//每次回溯，当len=0时也意味着着递归结束
	visited[i] = false;
}

void Map::Print_DFS(int i, int j)//输出两顶点间的所有路径和权值
{
	for (int i = 0; i < 999; i++) {
		path[i] = MAXINFNUM;
		visited[i] = false;
	}
	int len = 0;
	int sum = 0;
	bool f = false;
	DFS(i, j, path, len, sum, f);
	if (!f) {
		cout << "此路径在地图中不存在。" << endl;
	}
}

void Map::BFS(int start, int limit)//实现附近位置路线
{	
	cout << endl;
	for (int i = 0; i < 999; i++) {
		visited[i] = false;
		path[i] = -1; //存储当前顶点的上一个顶点
		dist[i] = MAXINFNUM;
	}
	queue<int> q; // 存储顶点的队列
	q.push(start); // 将起始顶点加入队列
	visited[start] = true; // 标记起始顶点已访问过
	//int distance[N]; // 存储每个顶点到起始顶点的距离
	//int prev[N]; // 存储当前顶点的上一个顶点
	dist[start] = 0; // 起始顶点到自身的距离为0

	while (!q.empty()) {
		int curr = q.front(); // 取出队头顶点
		q.pop();
		for (int i = 0; i < vernum; i++) {
			if (ArcArrary[curr][i] !=MAXINFNUM && !visited[i]) { // 当前顶点的相邻顶点未访问过且有边相连
				dist[i] = dist[curr] + 1; // 更新相邻顶点到起始顶点的距离
				path[i] = curr; // 记录上一个顶点
				if (dist[i] <= limit) { // 如果距离限制内，则将该顶点加入队列并标记为访问过
					q.push(i);
					visited[i] = true;
				}
			}
		}
	}

	// 输出所有符合条件的路径和距离
	for (int i = 0; i < vernum; i++) {
		if (visited[i] && dist[i] != 0 && dist[i] != MAXINFNUM) {
			cout << ">路径: " << arrary[start].get_data() << " -> " << arrary[i].get_data() << "    距离： " << dist[i] << endl;
			int curr = i;
			stack<int> s; // 存储路径的栈
			s.push(curr);
			while (curr != start) { // 从终点回溯到起点
				curr = path[curr]; // 获取上一个顶点
				s.push(curr);
			}
			string str = "";
			while (!s.empty()) { // 弹出栈中所有顶点，按正序输出路径
				str = str + arrary[s.top()].get_data() + "->";
				s.pop();
			}
			cout << str.substr(0, str.length() - 2);
			cout << endl;
		}
	}
}

void Map::Add_newVertex()//添加新的建筑物
{
	while (1)
	{
		cout << "输入新顶点：";
		string jianzhu = "", intr = "1";
		cin >> jianzhu;
		bool f = RepatGetData(jianzhu);//查看输入新顶点是否重复
		if (f) {
			cout << "目标已存在,请重新" << endl;
		}
		else {
			cout << "是否添加介绍信息? y :";//添加地点简介
			char c;
			cin >> c;
			if (c == 'y') {
				cout << "输入";
				cin >> intr;
			}	
			int eit = 0;
			database.Add_databases_place(eit,jianzhu.c_str(), intr.c_str());//存入数据库

			arrary[vernum].set_content(intr);//存入顶点数组
			arrary[vernum].set_data(jianzhu);
			arrary[vernum].set_id(eit);
			vernum++;
		}
		cout << "是否继续进行操作(y/n):";
		char ch;
		cin >> ch;
		if (ch == 'n')break;
	}
}

void Map::Add_newArc()//单独添加新边
{
	cout << "请输入由两个顶点构成的边" << endl;
	int m = 0, n = 0;
	string one = "", two = "";
	while (1) {
		cout << "顶点one：";
		cin >> one;
		m = GetDataIndex(one);//查找顶点在数组里面的下标
		cout << "顶点two：";
		cin >> two;
		n = GetDataIndex(two);
		if (m == -1 || n == -1) {
			cout << "目标顶点不存在" << endl;
		}
		else {
			//顶点下标，弧节点连接
			cout << "输入路径：";
			string wei = "";
			cin >> wei;
			while (!check(wei)) {//检查
				cout << "请输入正整数，不要输入负数或者小数或字母:";
				cin >> wei;
			}
			int we = atoi(wei.c_str());//强制转化
			ArcArrary[m][n] = ArcArrary[n][m] = we;//权值
			database.Add_databases_pathway(arrary[m].get_data(), arrary[n].get_data(), we);
		}
		char c = getchar();
		cout << "是否继续添加? y :";
		if (getchar() != 'y') break;
	}

}

void Map::Delete_oldVertex(string data)
{
	int i = GetDataIndex(data);//i为待删除顶点的序号
	if (i == -1) {
		cout << "未找到此数据。" << endl;
		return;
	}
	database.Delete_databases_place(arrary[i].get_data());//数据库删除
	cout << "相关 "<< arrary[i].get_data() <<" 的信息，全部删除成功。" << endl;
	for (int j = 0; j < vernum; j++)//删除与顶点i相连的边(如果有的话)
	{
		if (i != j) {
			Delete_oldArc(i, j);//如果存在此弧，则删除
		}
	}
	//排在顶点v后面的顶点的序号减1
	for (int j = i + 1; j < vernum; j++)
	{
		arrary[j - 1] = arrary[j];
	}
	vernum--; //顶点数减1
	
}

bool Map::Delete_oldArc(int i, int j)//删除原来的边
{
	ArcArrary[i][j] = ArcArrary[j][i] = MAXINFNUM;//邻接矩阵赋值无限大，代表是不可到达
	bool f = database.Delete_databases_patheay(arrary[i].get_data(), arrary[j].get_data());//数据库导入信息删除
	return f;
}

bool Map::Alter_ver(int i)
{
	bool f = false;
	string new_p = "";
	while (1) {
		if (vernum > 0) {
			cout << "地名为:" << arrary[i].get_data() << endl;
			cout << "请输入新的地名:";
			cin >> new_p;
			int p = GetDataIndex(new_p);
			if (p == -1 || i == p) {
				f = database.Alter_daatabases_ver(arrary[i].get_data(), new_p);
				//修改顶点数组
				arrary[i].set_data(new_p);
			}
			else {
				cout << "名字重复了。" << endl;
			}
		}
		else {
			cout << "表中无数据。" << endl;
		}
		char c = getchar();
		cout << "是否继续? y :";
		if (getchar() != 'y') break;
	}
	return f;
}

bool Map::Alter_arc(int i, int j)
{
	cout << "路径信息是:" << arrary[i].get_data() << " <--> " << arrary[j].get_data() << endl;
	cout << "（单路径)输入你要修改权值为:";
	string w = "";
	cin >> w;
	while (!check(w)) {//检查
		cout << "请输入正整数，不要输入负数或者小数或字母:";
		cin >> w;
	}
	int we = atoi(w.c_str());
	ArcArrary[i][j] = ArcArrary[j][i] = we;
	bool f2 = database.Alter_daatabases_arc(arrary[j].get_data(), arrary[i].get_data(), we);
	return f2;
}

void Map::Writing_information(int i)//描绘信息
{
	cout <<"地点:"<< arrary[i].get_data() << endl;
	if (arrary[i].get_content() == "1") {
		cout << "暂无介绍。" << endl;
	}
	else {
		cout << "内容:" << arrary[i].get_content() << endl;
	}
	string hau = "";
	cout << "请输入你要描绘的内容:";
	cin >> hau;
	char c = getchar();
	cout << "是否确定? y :";
	if (getchar() == 'y')
	{
		arrary[i].set_content(hau);
		database.Writing_inf(hau,arrary[i].get_id());
	}
	else {
		cout << "取消输入。" << endl;
	}
}

void Map::registered()
{
	system("cls");
	cout << "***注册  账号***" << endl;
	cout << "  1.管理员注册 2.退出" << endl;
	int c = 0;
	string ac = "", pa = "", ti = "";
	//随机数生成六位账号
	for (int i = 0; i < 6; ) {
		random_device seed;//硬件生成随机数种子
		ranlux48 engine(seed());//利用种子生成随机数引擎
		uniform_int_distribution<> distrib(0, 9);//设置随机数范围，并为均匀分布
		int random = distrib(engine);//随机数
		if (random == 0 && ac.length() == 0) {//确保第一位不是0
			continue;
		}
		ac += to_string(random);//生成的账号
		i++;
	}
	cout << "choice:";
	while (1) {
		cin >> c;
		if (c == 1)break;
		if (c == 2)return;
		else {
			cout << "格式错误。\nchoice:";
		}
	}
	while (1)
	{
		cout << "请输入超级权限口令：";//超及权限管理员拥有，密码 1
		string kl = "";
		cin >> kl;
		if (kl == "1") {
			cout << "*进入超级权限界面*" << endl;
			cout << "为你随机生成6位数字账号:" << ac << endl;
			time_t nowtime;
			while (1) {
				cout << "*设置密码:";
				cin >> pa;
				if (pa.length() > 8) {
					cout << "密码超出八位!请再次设置" << endl;//确保密码只能设置8位
					continue;
				}
				break;
			}
			//生成当地时间
			struct tm* p;;
			time(&nowtime);
			p = localtime(&nowtime);
			ti = "2023/" + to_string(p->tm_mon + 1) + "/" + to_string(p->tm_mday) + " " + to_string(p->tm_hour) + ":" + to_string(p->tm_min);
			cout << "*当前注册时间:" << ti << endl;
			database.Regisitered_database_manger(ti, ac, pa);
			system("pause");
			break;
		}
		else {
			cout << "格式错误。是否继续注册(y/n)：";
			string ch = "";
			cin >> ch;
			if (ch == "n")break;
		}
	}
}

bool Map::login(string& aCC)
{
	system("cls");
	cout << "***管理员  登录***" << endl;
	int i = 1;
	while (1) {
		char s;
		char ac[25], pa[25];
		int j = 0;//确保数组从0开始 
		cout << "*请输入账号：";
		cin >> ac;
		cout << "*请输入密码：";
		while ((s = _getch()) != '\r')
		{
			if (s == '\b')
			{
				if (j > 0) {
					j--;//回删一位 
					cout << "\b \b";
				}
			}
			else
			{
				pa[j] = s;  //存入数组 
				j++;
				cout << "*"; //用* 代替密码 
			}
		}
		pa[j] = '\0';//退出

		if (database.Login_database(ac,pa))//如果返回1，则表示查询到账号和密码，账号标注唯一建不可重复
		{
			cout << endl << "验证正确" << endl;
			for (int ij = 0; ij < 20; ij++)
			{
				cout << ".";
				Sleep(50);
			}
			aCC = ac;//记录账号，以便修改密码和注销操作
			return true;//登陆成功并选择进入什么操作界面
		}
		else
		{
			cout << endl << "*账号及密码错误!" << endl;
		}
		i++;
		if (i == 4)
		{
			cout << endl << ">已输入3次，退出登录" << endl;
			system("pause");
			return false;
		}
	}
}

void Map::modify(string acc)
{
	char a[25], pass[25];
	strcpy(a, acc.c_str());
	cout << "请输入旧密码:";
	string x = "";
	cin >> x;
	strcpy(pass, x.c_str());

	if (database.Login_database(a, pass)) {
		cout << "请输入新密码:";
		string pppa = "";
		cin >> pppa;
		database.Modify_database(acc, pppa);
		cout << "你已经将账号为 " << acc << " 用户密码修改为了 " << pppa << endl;
	}
	else {
		cout << "旧密码输入错误,请认真思考。" << endl;
	}
	system("pause");
}

void Map::logout(string acc)
{
	char a[25], pass[25];
	strcpy(a, acc.c_str());
	cout << "请输入旧密码:";
	string x = "";
	cin >> x;
	strcpy(pass, x.c_str());

	if (database.Login_database(a, pass)) {
		database.Logout_database(acc);
		cout << "账号 " << acc << " 已成功注销。" << endl;
	}
	else {
		cout << "旧密码输入错误,暂不可注销。" << endl;
	}
	system("pause");
}

```

### 3、用户

User.h

```cpp
#pragma once
#include<string>
#include"Databases.h"
using std::string;
class User
{
private:
	Databases user;//用户类调用数据库
private:
	string name = "";	//用户名
	string account = "";
	string password = "";
	string phone = "";//电话
	string email = "";//邮箱
	string r_time = "";//注册时间
public:
	void registered();//注册
	void login(string& ch, string& aCC);//登录
	void modify(string acc,string ch);//修改
	void logout(string acc,string ch);//注销
private:
	string get_name() {
		return name;
	}
	string get_account() {
		return account;
	}
	string get_password() {
		return password;
	}
	string get_phone() {
		return phone;
	}
	string get_email() {
		return email;
	}
	string get_time() {
		return r_time;
	}
	void set_name(string str) {
		this->name = str;
	}
	void set_account(string str) {
		this->account = str;
	}
	void set_password(string str) {
		this->password = str;
	}
	void set_phone(string str) {
		this->phone = str;
	}
	void set_email(string str) {
		this->email = str;
	}
	void set_time(string str) {
		this->r_time = str;
	}
};
```

User.cpp

```cpp
#define _CRT_SECURE_NO_WARNINGS
#include"User.h"
#include<iostream>
#include<string>
#include <random>//C++更加标准的随机数生成
#include<ctime>//获取当地时间
#include<conio.h>
#include<Windows.h>
using namespace std;
void User::registered()
{
	system("cls");
	cout << "***注册  账号***" << endl;
	cout << "  1.管理员注册 2.退出" << endl;
	int c = 0;
	string ac = "", pa = "",ti = "";
	//随机数生成六位账号
	for (int i = 0; i < 6; ) {
		random_device seed;//硬件生成随机数种子
		ranlux48 engine(seed());//利用种子生成随机数引擎
		uniform_int_distribution<> distrib(0, 9);//设置随机数范围，并为均匀分布
		int random = distrib(engine);//随机数
		if (random == 0 && ac.length() == 0) {//确保第一位不是0
			continue;
		}
		ac += to_string(random);//生成的账号
		i++;
	}
	while (1)
	{
		cin >> c;
		if (c == 2)
		{
			break;
		}
		else if (c == 1) 
		{
			cout << "请输入超级权限口令：";//超及权限管理员拥有，密码 1
			string kl = "";
			cin >> kl;
			if (kl == "1") {
				cout << "*进入超级权限界面*" << endl;
				cout << "为你随机生成6位数字账号:" << ac << endl;
				time_t nowtime;
				while (1) {
					cout << "*设置密码:";
					cin >> pa;
					if (pa.length() > 8) {
						cout << "密码超出八位!请再次设置" << endl;//确保密码只能设置8位
						continue;
					}
					break;
				}
				//生成当地时间
				struct tm* p;;
				time(&nowtime);
				p = localtime(&nowtime);
				ti = "2023/" + to_string(p->tm_mon + 1) + "/" + to_string(p->tm_mday) + " " + to_string(p->tm_hour) + ":" + to_string(p->tm_min);
				cout << "*当前注册时间:" << ti << endl;
				user.Regisitered_database_manger(ti, ac, pa);
				system("pause");
				break;
			}
		}
		else {
			cout << "格式错误。重新输入：";
		}
	}
}

void User::login(string &ch,string &aCC)//ch用来选择登录那种系统
{
	system("cls");
	if (ch == "1") {
		cout << "***管理员  登录***" << endl;
	}
	else {
		cout << "***用户  登录***" << endl;
	}
	int i = 1;
	while (1) {
		char s;
		char ac[25], pa[25];
		int j = 0;//确保数组从0开始 
		cout << "*请输入账号：";
		cin >> ac;
		cout << "*请输入密码：";
		while ((s = _getch()) != '\r')
		{
			if (s == '\b')
			{
				if (j > 0) {
					j--;//回删一位 
					cout << "\b \b";
				}
			}
			else 
			{  
				pa[j] = s;  //存入数组 
				j++;
				putchar('*'); //用* 代替密码 
			}
		}
		pa[j] = '\0';//退出
		int sign = 0;
		user.Login_database(ac, pa, sign, ch);
		if (sign == 1)//如果返回1，则表示查询到账号和密码，账号标注唯一建不可重复
		{
			aCC = ac;//记录账号，以便修改密码和注销操作
			cout << endl << "验证正确" << endl;
			for (int ij = 0; ij < 20; ij++)
			{
				cout << ".";
				Sleep(50);
			}
			//登陆成功并选择进入什么操作界面
			ch = "pass";
			return;
		}
		else
		{
			cout << endl << "*账号及密码错误!" << endl;
		}
		i++;
		if (i == 4)
		{
			cout << endl << ">已输入3次，退出登录" << endl;
			system("pause");
			break;
		}
	}
}

void User::modify(string acc,string ch)
{
	cout << "密码修改为：";
	string pppa = "";
	cin >> pppa;
	user.Modify_database(acc, pppa,ch);
}

void User::logout(string acc,string ch)
{
	cout << ch << endl;
	user.Logout_database(acc,ch);
}

```

## 报告+源码获取地址

详细内容请关注微信公众号：**GolangCode**，输入“**课程设计报告**” 获取详细内容。如果对你有小小的帮助，也请给我点个小赞赞。

![GolangCode](https://cdn.golangcode.cn/images/202501171944968.png)