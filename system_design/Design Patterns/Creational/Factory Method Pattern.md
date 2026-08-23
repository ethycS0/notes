Implement the Factory Method design pattern.

The Factory Method is a creational design pattern that provides an interface for creating objects in a superclass but allows subclasses to alter the type of objects that will be created.

You are given code that includes a few vehicles types and their respective factories. Complete the factory method implementation such that each factory returns the correct vehicle.

## Solution

The problem I had was I could not understand what the necessity of this pattern is. But now I understand a little atleast. Basically, think that there are some components that we require of a software project. These components may change from version to version (think different robots may require different sensor configurations). We dont want duplicate code. We also dont want, if the component changes or requires a new parameter, to copy paste the init or constructor throughout the codebase. Lets say we provide this assortment of components with a configuration file, we only need to make the decision once. Instead of creating the actual components, we create its factory only once. Then as and when we create the instances of this component using the factory throughout the codebase, runtime polymorphism will resolve the actual component created. This implementation solves all the above described issues. This method can be used for the following example requirements:

- Cross-Platform or Hardware Abstraction
- Plug-in Architectures and Extensible Frameworks
- Database Drivers / Data Access Layers
- UI Frameworks / Theme Engines (Dark Mode vs. Light Mode)
- Test Mocking and Unit Testing

```C++
class Vehicle {
   public:
    virtual string getType() = 0;
};

class Car : public Vehicle {
   public:
    string getType() override { return "Car"; }
};

class Bike : public Vehicle {
   public:
    string getType() override { return "Bike"; }
};

class Truck : public Vehicle {
   public:
    string getType() override { return "Truck"; }
};

class VehicleFactory {
   public:
    virtual Vehicle* createVehicle() = 0;
};

class CarFactory : public VehicleFactory {
    Vehicle* createVehicle() override { return new Car(); }
};

class BikeFactory : public VehicleFactory {
    Vehicle* createVehicle() override { return new Bike(); }
};

class TruckFactory : public VehicleFactory {
    Vehicle* createVehicle() override { return new Truck(); }
};
```