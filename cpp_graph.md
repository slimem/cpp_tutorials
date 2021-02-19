# CPP Graph algorithms

## Prim's minimum spanning tree with priority queue
Prim’s algorithm finds the cost of a minimum spanning tree from a weighted undirected graph.
This algorithm begins by randomly selecting a vertex and adding the least expensive edge from this vertex to the spanning tree.
The algorithm continues to add the least expensive edge from the vertices already added to the spanning tree to make it grow and terminates when all the vertices are added to the spanning tree.

With the following input, the following implementation displays ```15```:
```
5 6
1 2 3
1 3 4
4 2 6
5 2 2
2 3 5
3 5 7
1
```
```cpp
#include <bits/stdc++.h>

using namespace std;

using IP = std::pair<int,int>; // {weight, edge}

vector<string> split_string(string);

constexpr int N = 3e3+1; 

vector<IP> adj[N+1];
array<bool, N+1> visited;

int prims(int n, vector<vector<int>> edges, int start) { 
    priority_queue<IP, vector<IP>, greater<IP>> q;
    int cost = 0;
    
    // remember, starting node weight is always ZERO
    q.push(std::make_pair(0,start));
    
    while (!q.empty()) {
        
        auto p = q.top();
        q.pop();
        
        if (visited[p.second]) continue;
        
        cost += p.first;
        
        visited[p.second] = true;
    
        for (auto nodePair : adj[p.second]) {
            if (!visited[nodePair.second]) {
                q.push(nodePair);
            }
        }
    }
    
    return cost;
}

int main() {
    int n,e;
    cin>>n>>e;
    for(int i=0;i<e;i++)
    {
        int x,y,weight;
        cin>>x>>y>>weight;
        adj[x].push_back(make_pair(weight,y));
        adj[y].push_back(make_pair(weight,x));
    }
    int s;
    cin>>s;
    cout<<prim(s);
    
    return 0;
} 


```
