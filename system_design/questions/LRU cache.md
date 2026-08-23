```C++
class LRUCache {
   private:
    int max_capacity;

    // Key Value
    list<pair<int, int>> l;

    // Key Iterator
    unordered_map<int, list<pair<int, int>>::iterator> m;

   public:
    LRUCache(int capacity) : max_capacity(capacity) {}

    int get(int key) {
        auto it = m.find(key);
        if (it == m.end()) {
            return -1;
        }

        int value = it->second->second;

        l.erase(it->second);
        l.push_front({key, value});
        m[key] = l.begin();

        return value;
    }

    void put(int key, int value) {
        // Check Duplicates
        auto it = m.find(key);
        if (it != m.end()) {
            l.erase(it->second);
            l.push_front({key, value});
            m[key] = l.begin();

        // Check max capacity
        } else if (l.size() < max_capacity) {
            l.push_front({key, value});
            m.insert({key, l.begin()});
        } else {
            m.erase(prev(l.end())->first);
            l.pop_back();
            l.push_front({key, value});
            m.insert({key, l.begin()});
        }
    }
};
```

// Instead of:
// l.erase(it->second);
// l.push_front({key, value});
// m[key] = l.begin();

// Do this:
l.splice(l.begin(), l, it->second); // Moves existing node to front in O(1)

// Instead of:
m.erase(prev(l.end())->first);

// Do this:
m.erase(l.back().first);