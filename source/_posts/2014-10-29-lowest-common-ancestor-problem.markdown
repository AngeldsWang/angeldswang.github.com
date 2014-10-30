---
layout: post
title: "Lowest Common Ancestor Problem"
date: 2014-10-29 20:51
comments: true
categories: [Algorithm]
---

最近公共祖先问题(LCA)是一个很经典的树上的问题。目的就是找到距离2个节点最近的祖先节点。先抛开算法本身，这个问题有什么意义呢？[Alfred Aho，John Hopcroft，和Jeffrey Ullman](http://dl.acm.org/citation.cfm?id=804056)这几位大佬在70年代提出这些问题的时候勉为其难地想到了这样一个无聊的应用--["宗谱服务"](http://hihocoder.com/problemset/problem/1062)。大意就是有一个在线网站不断的获得一些家族成员之间的祖辈信息，并对任意两个成员提供查询他们最近公共祖先的服务。
<!--more-->
其实在70年代的时候，编译器设计优化是十分前沿的研究方向，很多算法都是为此服务的。这个最近公共祖先的问题实际上是作为编译器设计中一个很经典的问题--[在有向图上查找Dominator节点](<http://en.wikipedia.org/wiki/Dominator_(graph_theory)>)的一个子问题被引起广泛关注。不能再扯更多了，其他的应用比如版本控制什么的可以详见[stackoverflow上的讨论](http://stackoverflow.com/questions/3542452/what-are-the-practical-applications-of-the-lowest-common-ancestor-algorithms)。

##朴素解法
一个具体的问题如["宗谱服务"](http://hihocoder.com/problemset/problem/1062)
只需要从一个成员出发向上查找他的祖先(这里本人也是自己的祖先)，对找到的祖先进行标记，直到找不到更老的祖先为止。这样当另一个成员进行同样的向上查找时，遇到第一个已经标记过的祖先即为他们两个人的最近公共祖先。


``` cpp
/*	http://hihocoder.com/problemset/problem/1062
--------------------------
example input:
11
JiaYan JiaDaihua
JiaDaihua JiaFu
JiaDaihua JiaJing
JiaJing JiaZhen
JiaZhen JiaRong
JiaYuan JiaDaishan
JiaDaishan JiaShe
JiaDaishan JiaZheng
JiaShe JiaLian
JiaZheng JiaZhu
JiaZheng JiaBaoyu
3
JiaBaoyu JiaLian
JiaBaoyu JiaZheng
JiaBaoyu LinDaiyu
--------------------------
expected output:
JiaDaishan
JiaZheng
-1
--------------------------
*/

#include <iostream>
#include <map>
#include <string>

using namespace std;

typedef map<string, string> hashStrStr;
typedef map<string, bool> hashStrBool;

hashStrStr relations;

string query(hashStrStr relations, string person1, string person2) {
    string comFather;
    hashStrBool found;
    found[person1] = true;
    while (relations.find(person1) != relations.end()) {
        person1 = relations[person1];
        found[person1] = true;
    }

    if (found[person2] == true) {
        return person2;
    }

    while (relations.find(person2) != relations.end()) {
        person2 = relations[person2];
        if (found[person2] == true) {
            comFather = person2;
            break;
        }
    }

    return comFather;
}

int main() {
    int N, M;
    cin >> N;
    for (int i = 0; i < N; ++i) {
        string father, son;
        cin >> father >> son;
        relations[son] = father;
    }
    cin >> M;
    string per1, per2;
    for (int i = 0; i < M; ++i) {
        cin >> per1 >> per2;
        string comFather = query(relations, per1, per2);
        if (!comFather.empty()) {
            cout << comFather << endl;
        } else {
            cout << -1 << endl;
        }
    }
    return 0;
}
```

##Tarjan算法(off-line)
上述朴素算法的复杂度是$$\mathcal{O}(N)$$，但是这是对于一次查询而言。如果数据$$N$$很多，且查询次数$$M$$很大，$$\mathcal{O}(NM)$$复杂度对于[在线查询服务](http://hihocoder.com/problemset/problem/1067)来说就是难以接受的。Tarjan算法可以把复杂度缩减到$$\mathcal{O}(N+M)$$。Tarjan是一种离线算法，也就是说事先收集好所有的Query。而真正的实现部分是由[DFS+并查集](http://blog.csdn.net/jarjingx/article/details/8183240)实现的。


``` cpp
/*	http://hihocoder.com/problemset/problem/1067
--------------------------
example input:
4
Adam Sam
Sam Joey
Sam Micheal
Adam Kevin
3
Sam Sam
Adam Sam
Micheal Kevin
--------------------------
expected output:
Sam
Adam
Adam
--------------------------
*/

#include <iostream>
#include <vector>
#include <map>

using namespace std;

const int MAXN = 100001;

map<string, int> getIdx;
map<int, string> getName;

vector<int> adjList[MAXN];
pair<string, string> queries[MAXN];
map<int, vector<int> > relativeQueries;
int inDegree[MAXN];
int ufs[MAXN * 2];
string ancestor[MAXN];
bool visited[MAXN * 2];


void addEdge(string father, string son) {
    adjList[getIdx[father]].push_back(getIdx[son]);
    inDegree[getIdx[son]]++;
}

void ufs_init() {
    for (int i = 0; i < MAXN * 2; ++i) {
        ufs[i] = i;
    }
}

int ufs_find_root(int x) {
    if (x == ufs[x]) return x;
    return ufs[x] = ufs_find_root(ufs[x]);
}

void ufs_union(int x, int y) {
    const int rx = ufs_find_root(x);
    const int ry = ufs_find_root(y);
    if (rx == ry) return;
    ufs[ry] = rx;
}

int dfs(int x) {
    for (int i = 0; i < adjList[x].size(); ++i) {
        dfs(adjList[x][i]);
        ufs_union(x, adjList[x][i]);
    }
    visited[x] = true;
    for (int i = 0; i < relativeQueries[x].size(); ++i) {
        int queryIdx = relativeQueries[x][i];
        if (ancestor[queryIdx] != "") continue;
        if (queries[queryIdx].first == queries[queryIdx].second) {
            ancestor[queryIdx] = queries[queryIdx].first;
        } else if (getIdx[queries[queryIdx].first] == x && 
                visited[getIdx[queries[queryIdx].second]]) {
            ancestor[queryIdx] = getName[ufs_find_root(getIdx[queries[queryIdx].second])];
        } else if (getIdx[queries[queryIdx].second] == x && 
                visited[getIdx[queries[queryIdx].first]]) {
            ancestor[queryIdx] = getName[ufs_find_root(getIdx[queries[queryIdx].first])];
        }
    }
}


int main() {
    int N, M;
    cin >> N;
    int count = 0;
    for (int i = 0; i < N; ++i) {
        string father, son;
        cin >> father >> son;
        if (getIdx.find(father) == getIdx.end()) {
            getIdx[father] = count;
            getName[count] = father;
            count++;
        }
        if (getIdx.find(son) == getIdx.end()) {
            getIdx[son] = count;
            getName[count] = son;
            count++;
        }
        addEdge(father, son);
    }
    int rootIdx;
    for (int i = 0; i < count; ++i) {
        if (inDegree[i] == 0) {
            rootIdx = i;
            break;
        }
    }
    cin >> M;
    ufs_init();
    for (int i = 0; i < M; ++i) {
        string f, s;
        cin >> f >> s;
        relativeQueries[getIdx[f]].push_back(i);
        relativeQueries[getIdx[s]].push_back(i);
        queries[i] = make_pair(f, s);
    }
    dfs(rootIdx);
    for (int i = 0; i < M; ++i) {
        cout << ancestor[i] << endl;
    }
    return 0;
}
```

##转化成RMQ(on-line)
离线算法虽然高效，但是有一个明显的问题。因为实际应用中，我们不可能提前搜集好所有的Query。那么有没有什么在线的算法同时保持高效的查询呢。当然是有的，RMQ就可以做到，虽然正常人第一感觉这两个问题好像没有什么明显的关系。实际上，只需要简单的预先处理一下现有的祖辈关系，建立一颗多叉树，通过一次DFS遍历就可以将问题完全转化成RMQ。
实际上[RMQ还可以转化成LCA](http://noalgo.info/496.html)，这两者是可以互相转换的。

``` cpp
/*	http://hihocoder.com/contest/hiho17/problem/1
--------------------------
example input:
4
Adam Sam
Sam Joey
Sam Micheal
Adam Kevin
3
Sam Sam
Adam Sam
Micheal Kevin
--------------------------
expected output:
Sam
Adam
Adam
--------------------------
*/

#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <algorithm>
#include <climits>

using namespace std;

#define L(t) (t << 1)
#define R(t) (t << 1 | 1)

const int MAXN = 100001;
const int MAXM = 100001;

typedef struct node_t {
    int left, right;
    int min, minIdx;
}node_t;

int val[MAXN * 2];
int depth[MAXN * 2];
int first[MAXN];
int len = 1;
node_t nodes[MAXN * 4];
int minx;
int minxIdx;

map<string, int> getIdx;
map<int, string> getName;
vector<int> adjList[MAXN];
int inDegree[MAXN];

void addEdge(int father, int son) {
    adjList[father].push_back(son);
    inDegree[son]++;
}

void dfs(int idx, int dep) {
    depth[len] = dep;
    val[len] = idx;
    first[idx] = len;
    len++;
    for (int i = 0; i < adjList[idx].size(); ++i) {
        dfs(adjList[idx][i], dep + 1);
        depth[len] = dep;
        val[len] = idx;
        len++;
    }
}

void build_tree(int t, int l, int r) {
    nodes[t].left = l;
    nodes[t].right = r;
    if (l == r) {
        nodes[t].min = depth[l];
        nodes[t].minIdx = l;
        return;
    }
    const int mid = (l + r) / 2;
    build_tree(L(t), l, mid);
    build_tree(R(t), mid + 1, r);
    nodes[t].min = min(nodes[L(t)].min, nodes[R(t)].min);
    if (nodes[t].min == nodes[L(t)].min) {nodes[t].minIdx = nodes[L(t)].minIdx;}
    else if (nodes[t].min == nodes[R(t)].min) {nodes[t].minIdx = nodes[R(t)].minIdx;}
}

void query(int t, int l, int r) {
    if (nodes[t].left == l && nodes[t].right == r) {
        if (nodes[t].min < minx) {
            minx = nodes[t].min;
            minxIdx = nodes[t].minIdx;
        }
        return;
    }
    const int mid = (nodes[t].left + nodes[t].right) / 2;
    if (mid < l) {
        query(R(t), l, r);
    } else if (mid >= r) {
        query(L(t), l, r);
    } else {
        query(L(t), l, mid);
        query(R(t), mid + 1, r);
    }
}

int main() {
    int N, M;
    cin >> N;
    int count = 0;
    for (int i = 0; i < N; ++i) {
        string per1, per2;
        cin >> per1 >> per2;
        if (getIdx.find(per1) == getIdx.end()) {
            getIdx[per1] = count;
            getName[count] = per1;
            count++;
        }
        if (getIdx.find(per2) == getIdx.end()) {
            getIdx[per2] = count;
            getName[count] = per2;
            count++;
        }

        addEdge(getIdx[per1], getIdx[per2]);
    }
    int rootIdx;
    for (int i = 0; i < MAXN * 2; ++i) {
        if (inDegree[i] == 0) {
            rootIdx = i;
            break;
        }
    }
    dfs(rootIdx, 0);
    build_tree(1, 1, len - 2);
    cin >> M;
    for (int i = 0; i < M; ++i) {
        string p1, p2;
        cin >> p1 >> p2;
        int l, r;
        l = min(first[getIdx[p1]], first[getIdx[p2]]);
        r = max(first[getIdx[p1]], first[getIdx[p2]]);
        minx = INT_MAX;
        query(1, l, r);
        cout << getName[val[minxIdx]] << endl;
    }
    return 0;
}
```




