Implement the State design pattern.

The State is a behavioral design pattern that lets an object alter its behavior when its internal state changes. It appears as if the object changed its class.

You are given a Document context class and an abstract State interface. A document can transition between states: DraftState -> ModerationState -> PublishedState.

You need to implement the following:

- The publish() method in DraftState to transition the document's state to ModerationState.
- The publish() method in ModerationState to transition the document's state to PublishedState.
- The publish() method in PublishedState where the document is already published.

## Solution

This is intersting usage of classes. It seems that there is some circular usage of these classes. Essentailly as I understand, that internal pointer state of an object can be switched between various child classes of state. Because the state machine exists in the state class, we call the current state state to change for the object. This internally calls the state change in itself that change the cild class. Very interesting loop.

- State has a method to change state
- Non-Abstract class has a pointer of the instance state and a method to change state, which internally calls the instance state->change_state method.
- Now we declare multiple states that is State children and override the change state method to change to the next.

This kinda localizes the behaviour of the Non-Abstract object to a state. So allows something like a game character to perform different actions in different states.

```C++
class Document;

class State {
   public:
    virtual ~State() {}
    virtual void publish(Document* doc) = 0;
};

class Document {
   private:
    State* state;

   public:
    Document(State* initialState) : state(initialState) {}

    void setState(State* newState) {
        state = newState;
    }

    void publish() {
        state->publish(this);
    }
};

class DraftState : public State {
   public:
    void publish(Document* doc) override {
		doc->setState(new ModerationState);
    }
};

class ModerationState : public State {
   public:
    void publish(Document* doc) override {
		doc->setState(new PublishedState());
    }
};

class PublishedState : public State {
   public:
    void publish(Document* doc) override {
		std::cout << "Document is already published. Cannot publish further.\n";
    }
};
```
