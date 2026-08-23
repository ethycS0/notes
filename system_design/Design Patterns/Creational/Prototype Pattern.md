Implement the Prototype design pattern.

The Prototype is a creational design pattern that allows an object to copy itself. It is particularly useful when the creation of an object is more convenient through copying an existing object than through creation from scratch.

You are given a Shape prototype interface that supports cloning itself within the Rectangle and Square classes.

You need to implement the following:

- The clone() method in the Rectangle class which returns a new instance of the Rectangle class.
- The clone() method in the Square class which returns a new instance of the Square class.
- The cloneShapes() method in the Test class which returns a list of cloned shapes.

## Solution

This pattern is more intuitive. We clone instead of creating a new object. This can be useful for the following things:
- Heavy Initialization / Expensive Database or Network Calls
- Avoiding Subclass Explosion
- Undo/Redo Functionality (State History)
- Preserving Encapsulation (Hiding Constructor Details)

```C++
class Shape {
   public:
    virtual ~Shape() {}
    virtual Shape* clone() const = 0;
};

class Rectangle : public Shape {
   private:
    int width;
    int height;

   public:
    Rectangle(int w, int h) : width(w), height(h) {}

    int getWidth() const { return width; }

    int getHeight() const { return height; }

    Shape* clone() const override { return new Rectangle(width, height); }
};

class Square : public Shape {
   private:
    int length;

   public:
    Square(int l) : length(l) {}

    int getLength() const { return length; }

    Shape* clone() const override { return new Square(length); }
};

class Test {
   public:
    vector<Shape*> cloneShapes(const vector<Shape*>& shapes) {
        vector<Shape*> result;
        for (auto& s : shapes) {
            result.push_back(s->clone());
        }

        return result;
    }
};
```