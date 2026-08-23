Implement the Strategy design pattern.

The Strategy is a behavioral design pattern that lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

You are given a PaymentStrategy interface and a ShoppingCart context class.

You need to implement the following:

- The pay(int amount) method in the CreditCardPayment class.
- The pay(int amount) method in the PayPalPayment class.
- The checkout(int amount) method in the ShoppingCart class to execute the configured strategy.

## Solution

This seems optimal usage of OOPs for me. Instead of having a massve if else for a choice (of payment in this case), we can have the runtime call the correct function. This simply requires a Base Class for the strategy and children classes of all the applicable choices. Then we can just set the required object ad call pay so runtime calls teh correct function. 

```C++
class PaymentStrategy {
   public:
    virtual ~PaymentStrategy() {}
    virtual void pay(int amount) = 0;
};

class CreditCardPayment : public PaymentStrategy {
   private:
    std::string cardNumber;

   public:
    CreditCardPayment(const std::string& cardNum) : cardNumber(cardNum) {}

    void pay(int amount) override {
		std::cout << "Card Number: " << cardNumber << std::endl;
		std::cout << "Amount: " << amount << std::endl;
		std::cout << "Payment Successfull" << std::endl;
    }
};

class PayPalPayment : public PaymentStrategy {
   private:
    std::string email;

   public:
    PayPalPayment(const std::string& emailAcc) : email(emailAcc) {}

    void pay(int amount) override {
		std::cout << "Account Email: " << email << std::endl;
		std::cout << "Amount: " << amount << std::endl;
		std::cout << "Payment Successfull" << std::endl;
    }
};

class ShoppingCart {
   private:
    PaymentStrategy* paymentStrategy;

   public:
    ShoppingCart() : paymentStrategy(nullptr) {}

    void setPaymentStrategy(PaymentStrategy* strategy) {
        paymentStrategy = strategy;
    }

    void checkout(int amount) {
		paymentStrategy->pay(amount);
    }
};
```
