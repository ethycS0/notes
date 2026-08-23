Implement the Decorator design pattern.

The Decorator is a structural design pattern that lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.

You are given a base Coffee interface, a SimpleCoffee concrete implementation, and an abstract CoffeeDecorator class.

You need to implement the following:

- The MilkDecorator class which adds cost (+0.50) and updates the description (adds ", Milk").
- The SugarDecorator class which adds cost (+0.25) and updates the description (adds ", Sugar").

## Solution

Structural Patterns are quite more intuitive. We can always find us in a solution where we require a method/s for an objects interaction in only 1 place. Instead of bloating the class or creating a subclasses nightmare (all iterations of Simple Coffee Milk and Sugar so 4 subclasses) for this 1 instance of usage only, we can use this Decorator pattern to attach a special behavious for that instance. This sounds quite simple and intuitive.

```C++
class Coffee {
   public:
    virtual ~Coffee() {}
    virtual std::string getDescription() const = 0;
    virtual double getCost() const = 0;
};

class SimpleCoffee : public Coffee {
   public:
    std::string getDescription() const override { return "Simple Coffee"; }
    double getCost() const override { return 2.00; }
};

class CoffeeDecorator : public Coffee {
   protected:
    Coffee* coffee;

   public:
    CoffeeDecorator(Coffee* c) : coffee(c) {}
};

class MilkDecorator : public CoffeeDecorator {
   public:
    MilkDecorator(Coffee* c) : CoffeeDecorator(c) {}

    std::string getDescription() const override {
        return (coffee->getDescription() + ", Milk");
    }

    double getCost() const override {
        return (coffee->getCost() + 0.50);
    }
};

class SugarDecorator : public CoffeeDecorator {
   public:
    SugarDecorator(Coffee* c) : CoffeeDecorator(c) {}

    std::string getDescription() const override {
        return (coffee->getDescription() + ", Sugar");
    }

    double getCost() const override {
        return (coffee->getCost() + 0.25);
    }
};
```
