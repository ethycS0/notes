Previous: [[Task Scheduler (M) TODO]] | Next: [[Breadth First Traversal of a Graph (LOL) TODO]]

https://leetcode.com/problems/number-of-islands/

Given a 2D grid grid where '1' represents land and '0' represents water, count and return the number of islands.

An island is formed by connecting adjacent lands horizontally or vertically and is surrounded by water. You may assume water is surrounding the grid (i.e., all the edges are water). 

## Solution

BFS

- Get an adjacent array{[i-1, j], [i, j-1], [i+1, j], [i, j+1]} and empty visited array
- Start at [0, 0], check if 1 and append to visited
- Iterate i j till we find a 1
- When we do, increment num_islands, get valid {not yet visited and within bounds} adjacents, add them to visited array and add adjacents that are 1 to check array. Here we start the BFS
- Repeat the steps per point in check array: get valid adjacents, add to visited and add adjacents that are 1 to check array
- When check array is empty, iterate over remaining non-visited points, if 1 is found again repeat above steps


```C++
class Solution {
   public:
    int numIslands(vector<vector<char>>& grid) {
        vector<vector<int>> adjacent({{-1, 0}, {0, -1}, {1, 0}, {0, 1}});

        set<vector<int>> visited;
        vector<vector<int>> search;

        int num_islands = 0;
        int n = grid.size();
        int m = grid[0].size();

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (visited.contains({i, j}) == true) {
                    continue;
                }

                visited.insert({i, j});

                if (grid.at(i).at(j) == '1') {
                    num_islands += 1;
                    search.push_back({i, j});

                    while (!search.empty()) {
                        vector<int> cords = search.back();
                        search.pop_back();
                        for (auto a : adjacent) {
                            int x = cords.at(0) + a.at(0);
                            int y = cords.at(1) + a.at(1);

                            if (x > -1 && x < n && y > -1 && y < m && !visited.contains({x, y})) {
                                if (grid.at(x).at(y) == '1') {
                                    search.push_back({x, y});
                                    visited.insert({x, y});
                                }
                            }
                        }
                    }
                }
            }
        }
        return num_islands;
    }
};

```
