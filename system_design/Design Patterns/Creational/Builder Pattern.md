Implement the Builder design pattern.

The Builder is a creational design pattern that lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.

You are given a Computer class with multiple optional components (CPU, RAM, Storage, GPU) and a ComputerBuilder interface.

You need to implement the following:

- The step-by-step configuration methods (setCPU, setRAM, setStorage, setGPU) in the DesktopBuilder class, enabling method chaining.
- The build() method in DesktopBuilder to assemble and return the fully constructed Computer instance.

## Solution

So I was so confused on what is actually going on? What does the Computer class itself not have a contructor to set all the values? Apparently it is to handle various configurations where some are optional and might need multiple constructor overloads. Even then, we can just pass default values? (Telescoping). That is actually normal, but what if there are too many input values? Then what? So in that case we have do this builder pattern. It also allows a more chaining and readable initialization making configurations clear. It also allows polymorphism for vaious types like Computer vs Laptop.

```C++
class Computer {
   public:
    std::string cpu;
    std::string ram;
    std::string storage;
    std::string gpu;

    void display() const {
        std::cout << "CPU: " << cpu << ", RAM: " << ram
                  << ", Storage: " << storage << ", GPU: " << gpu << std::endl;
    }
};

class ComputerBuilder {
   public:
    virtual ~ComputerBuilder() {}
    virtual ComputerBuilder& setCPU(const std::string& cpu) = 0;
    virtual ComputerBuilder& setRAM(const std::string& ram) = 0;
    virtual ComputerBuilder& setStorage(const std::string& storage) = 0;
    virtual ComputerBuilder& setGPU(const std::string& gpu) = 0;
    virtual Computer build() = 0;
};

class DesktopBuilder : public ComputerBuilder {
   private:
    Computer computer;

   public:
    DesktopBuilder() {}

    ComputerBuilder& setCPU(const std::string& cpu) override {
			computer.cpu = cpu;
        return *this;
    }

    ComputerBuilder& setRAM(const std::string& ram) override {
			computer.ram = ram;
        return *this;
    }

    ComputerBuilder& setStorage(const std::string& storage) override {
			computer.storage = storage;
        return *this;
    }

    ComputerBuilder& setGPU(const std::string& gpu) override {
			computer.gpu = gpu;
        return *this;
    }

    Computer build() override {
        return computer;
    }
};
```
