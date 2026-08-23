Implement the Adapter design pattern.

The Adapter is a structural design pattern that allows incompatible interfaces to work together. It wraps an existing class with a new interface so that it becomes compatible with the client's interface.

You are given completed SquareHole, Square and Circle classes. A Square fits into a SquareHole if the Square's side length is less than or equal to the SquareHole's length. A Circle has a radius and a Circle fits into a SquareHole if the Circle's diameter is less than or equal to the SquareHole's length.

Complete the implementation of the CircleToSquareAdapter class such that it adapts a Circle to a Square.

## Solution

This pattern is much more intuitive. Basically, there are 2 interfaces that are not compatible or another situation like an output type is different than the required one, we can use an adapter. Similar to how a type-C -> type-A adapter works, an adapter class will be the wrapper around the input class and create the required output. This allows 2 different interfaces to work together. 

```C++
class Square {
    double sideLength;

   public:
    Square() : sideLength(0) {}
    Square(double sideLength) : sideLength(sideLength) {}

    virtual double getSideLength() { return sideLength; }
};

class SquareHole {
    double sideLength;

   public:
    SquareHole(double sideLength) : sideLength(sideLength) {}

    bool canFit(Square& square) { return sideLength >= square.getSideLength(); }
};

class Circle {
    double radius;

   public:
    Circle(double radius) : radius(radius) {}

    double getRadius() { return radius; }
};

class CircleToSquareAdapter : public Square {
   public:
    CircleToSquareAdapter(Circle& circle) : Square(circle.getRadius() * 2) {}

    double getSideLength() override { return Square::getSideLength(); }
};

```