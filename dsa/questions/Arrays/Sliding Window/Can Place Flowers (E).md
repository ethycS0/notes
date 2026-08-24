Previous: [[Maximum Average Subarray I (E)]] | Next: [[Kids With the Greatest Number of Candies (E)]]

https://leetcode.com/problems/can-place-flowers/

You have a long flowerbed in which some of the plots are planted, and some are not. Given an integer array  containing 's and 's, and an integer , return  if  new flowers can be planted in the flowerbed without violating the no-adjacent-flowers rule.

## Solution
```C++
class Solution {
public:
    bool canPlaceFlowers(vector<int>& flowerbed, int n) {
        int length = flowerbed.size();
        int planted = 0;

        // Size =1
        if (length == 1) {
            if (flowerbed[0] == 0) {
                planted = 1;
            }
            if (planted >= n) {
                return true;
            } else {
                return false;
            }
        }

        // Size = 2
        if (length == 2) {
            if (flowerbed[0] == 0 && flowerbed[1] == 0) {
                planted = 1;
            }
            if (planted >= n) {
                return true;
            } else {
                return false;
            }
        }

        // If Starting edge has 2 zeroes
        if (flowerbed[0] == 0 && flowerbed[1] == 0) {
            planted += 1;
        }

        // If Ending edge has 2 zeroes
        if (flowerbed[length - 1] == 0 && flowerbed[length - 2] == 0) {
            planted += 1;
        }

        // If there are 3 consecutive zeroes
        if (length > 4) {
            for (int i = 1; i < length - 3; i++) {
                for (int j = 0; j < 3; j++) {
                    if (flowerbed[i + j] != 0) {
                        break;
                    }

                    if (j == 2) {
                        planted += 1;
                        i += 1;
                    }
                }
            }
        }

        if (planted >= n) {
            return true;
        } else {
            return false;
        }
    }
};
```