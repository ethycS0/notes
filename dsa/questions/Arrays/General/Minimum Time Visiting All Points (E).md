Previous: [[How Many Numbers Are Smaller Than the Current Number (E)]] | Next: [[Spiral Matrix (M)]]

On a 2D plane, there are n points with integer coordinates points[i] = [xi, yi]. Return the minimum time in seconds to visit all the points in the order given by points.

You can move according to these rules:

    In 1 second, you can either:
        move vertically by one unit,
        move horizontally by one unit, or
        move diagonally sqrt(2) units (in other words, move one unit vertically then one unit horizontally in 1 second).
    You have to visit the points in the same order as they appear in the array.
    You are allowed to pass through points that appear later in the order, but these do not count as visits.


## Solution

- Linear: For a point pair(i, i+1), do x = abs(x2 - x1) and abs(y = y2 - y1), then do min(x, y) + abs(x - y) -> travelling time for those pairs. Do for all.

```C++
class Solution {
public:
    int minTimeToVisitAllPoints(vector<vector<int>>& points) {
        int time = 0;
        for(int i = 0; i < points.size() - 1; i++) {
            vector<int> curr_location = points[i];
            vector<int> next_location = points[i + 1];

            int max_x = next_location[0] - curr_location[0];
            int max_y = next_location[1] - curr_location[1];

            int traversal = abs(max_y) - abs(max_x);

            if(traversal == 0) {
                time += abs(max_y);
            } else if(traversal > 0) {
                time += abs(max_x);
                time += abs(traversal);
            } else if(traversal < 0) {
                time += abs(max_y);
                time += abs(traversal);
            }
        }

        return time;
    }
};
```