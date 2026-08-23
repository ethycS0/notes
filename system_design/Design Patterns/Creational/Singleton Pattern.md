Implement the Singleton design pattern.

The Singleton is a creational design pattern that lets you ensure that a class has only one instance, while providing a global access point to this instance.

You are given code for a Singleton class intended to manage application configuration settings. You need to ensure only a single instance can exist and provide thread-safe access to it.

You need to implement the following:

- Prevent copy construction and assignment.
- The getInstance() method to return the single instance of the Singleton class.
- The configuration state getter and setter methods.

## Solution

This pattern is used if there should only ever be 1 singular instance of an object. 
- Logging System
- Configuration Managers
- Thread Pools

So a singular instance that can be taken everywhere in the code. This requires us to create the instance as a static member of the class with a private constructor. Then we require a static factory method that would provide the instance globablly. Additional steps can be taken to switch from lazy initialization to mutex guarded and deleting copy constructors. C++11 also adds something called Meyers Singleton that maintains a better thread safety.

```C++
class Singleton {
   private:
    std::string configValue;
    static Singleton* instance;

    Singleton() : configValue("Default Configuration") {}

   public:
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton* getInstance() {
		if (instance == nullptr) {
			instance = new Singleton();
		}
		return instance;
    }

    std::string getValue() const {
        return instance->configValue;
    }

    void setValue(const std::string& value) {
		instance->configValue  = value;
    }
};

Singleton* Singleton::instance = nullptr;
```
