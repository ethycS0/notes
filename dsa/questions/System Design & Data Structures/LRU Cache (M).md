Previous: [[Reverse First K elements of a Queue (LOL)]] | Next: [[Binary Search (E)]]

https://leetcode.com/problems/lru-cache/

Design a data structure that follows the constraints of a Least Recently Used (LRU) cache, supporting  and  operations in O(1) average time complexity.

## Solution

```C++
class LRUCache {
private:
    int max_capacity;

    list<pair<int, int>> storage;
    unordered_map < int, list<pair<int, int>>::iterator> cache;

public:
    LRUCache(int capacity) : max_capacity(capacity) {}

    int get(int key) {
        if (cache.find(key) == cache.end()) {
            return -1;
        }

        // Get iterator
        auto it = cache[key];

        // Get pair at iterator from storage
        pair<int, int> data = *it;

        // Delete that iterator
        storage.erase(it);

        // Push_back the pair again in storage
        storage.push_back(data);

        // Update the cache iterator
        cache[key] = prev(storage.end());

        // Return value
        return data.second;
    }

    void put(int key, int value) {
        // Found the key
        if (cache.find(key) != cache.end()) {
            // Delete Previous Entry
            storage.erase(cache[key]);

            // Store new data
            storage.push_back({key, value});

            // Update Cache Entry
            cache[key] = prev(storage.end());

            return;
        }

        // New data
        // Capacity Filled
        if (storage.size() >= max_capacity) {
            // Get LRU data entry
            pair<int, int> data = storage.front();

            // Remove LRU data
            storage.erase(storage.begin());

            // Evict LRU key from cache
            cache.erase(data.first);
        }

        // Store new data in storage
        storage.push_back({key, value});

        // Store new data in cache
        cache.insert({key, prev(storage.end())});
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```