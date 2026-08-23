Implement the Facade design pattern.

The Facade is a structural design pattern that provides a simplified interface to a library, a framework, or any other complex set of classes.

You are given completed subsystem classes: Amplifier, Lights, and Projector.

You need to implement the following:

- The watchMovie() method in SmartHomeFacade to orchestrate turning on and setting up all subsystems.
- The endMovie() method in SmartHomeFacade to turn off and reset all subsystems.

## Solution

This pattern happens a lot. This is easy to understand no requirement of explanation.

```C++
class Amplifier {
   public:
    void turnOn() { std::cout << "Amplifier turned ON\n"; }
    void turnOff() { std::cout << "Amplifier turned OFF\n"; }
    void setVolume(int level) { std::cout << "Amplifier volume set to " << level << "\n"; }
};

class Lights {
   public:
    void dim(int level) { std::cout << "Lights dimmed to " << level << "%\n"; }
    void turnOn() { std::cout << "Lights turned ON full brightness\n"; }
};

class Projector {
   public:
    void turnOn() { std::cout << "Projector turned ON\n"; }
    void turnOff() { std::cout << "Projector turned OFF\n"; }
    void setInputHDMI() { std::cout << "Projector input set to HDMI\n"; }
};

class SmartHomeFacade {
   private:
    Amplifier amp;
    Lights lights;
    Projector projector;

   public:
    SmartHomeFacade(Amplifier a, Lights l, Projector p)
        : amp(a), lights(l), projector(p) {}

    void watchMovie() {
		lights.dim(10);
		projector.turnOn();
		projector.setInputHDMI();
		amp.turnOn();
		amp.setVolume(5);
    }

    void endMovie() {
		amp.turnOff();
		projector.turnOff();
		lights.dim(100);
    }
};
```
