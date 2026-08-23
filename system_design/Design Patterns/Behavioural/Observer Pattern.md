Implement the Observer design pattern.

The Observer is a behavioral design pattern that lets you define a subscription mechanism to notify multiple objects about any events that happen to the object they're observing.

You are given an Observer interface and a Subject interface.

You need to implement the following:

- The attach(Observer*) method in WeatherStation to add a new observer.
- The detach(Observer*) method in WeatherStation to remove an observer.
- The notifyObservers() method in WeatherStation to notify all attached observers of a temperature update.
- The update(float) method in PhoneDisplay to handle temperature updates.

## Solution

This is a pretty standard event notifier system example. The observer pattern allows multiple entities to subscribe or attach to events. Events are basically a computational occurance attached to a list of subscribers. When that occurance triggers, all the subscribers are notified using callback functions. We must allows addition and removal of subscribers from the event list.

```C++
class Observer {
   public:
    virtual ~Observer() {}
    virtual void update(float temp) = 0;
};

class Subject {
   public:
    virtual ~Subject() {}
    virtual void attach(Observer* observer) = 0;
    virtual void detach(Observer* observer) = 0;
    virtual void notifyObservers() = 0;
};

class WeatherStation : public Subject {
   private:
    std::vector<Observer*> observers;
    float temperature;

   public:
    void attach(Observer* observer) override {
		observers.push_back(observer);
    }

    void detach(Observer* observer) override {
		auto it = std::find(observers.begin(), observers.end(), observer);
		if (it != observers.end()) {
			observers.erase(it);
		}
    }

    void notifyObservers() override {
		for(auto& o : observers) {
			o->update(temperature);
		}
    }

    void setTemperature(float temp) {
        temperature = temp;
        notifyObservers();
    }
};

class PhoneDisplay : public Observer {
   private:
    float currentTemp;

   public:
    PhoneDisplay() : currentTemp(0.0f) {}

    void update(float temp) override {
		currentTemp = temp;
    }
};
```
