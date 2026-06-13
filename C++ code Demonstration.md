# C++ Code Demonstration: SOLID Principles

## Overview
This document demonstrates SOLID Principles through a Food Order Service example.

The Food Order Service includes the following functionality:
- Select menu items and add them to the cart
- Calculate the total price
- Apply discounts
- Process payments
- Send order notifications
---

## Code Implementation

```cpp
#include<iostream>
#include<vector>

using namespace std;

//separate classes for each responsibility i.e Single Responsibility Principle

class CartItem {
    public :
        string item;
        int quntity;
        double price;

        CartItem() {
            this->item = "";
            this->quntity = 0;
            this->price = 0.0;
        }

        CartItem(string item, int quntity, double price) {
            this->item = item;
            this->quntity = quntity;
            this->price = price;
        }

        void setItem(string item) {
            this->item = item;
        }
        void setQuntity(int quntity) {
            this->quntity = quntity;
        }
        void setPrice(double price) {
            this->price = price;
        }
        string getItem() {
            return this->item;
        }
        int getQuntity() {
            return this->quntity;
        }
        double getPrice() {
            return this->price;
        }
};

class ItemSelector {
    private:
        vector<CartItem> items;
    public:
        
        vector<CartItem> selectItems() {
            items.push_back(CartItem("Pizza", 2, 200.0));
            items.push_back(CartItem("Burger", 2, 100.0));

            cout<<"Items in cart " << items.size() <<endl;
            return items;
        }

};

class PriceCalculator {
    public:
        double calculatePrice(vector<CartItem> items) {
            double totalPrice = 0.0;
            for(int i=0; i<items.size(); i++) {
                totalPrice += items[i].getQuntity() * items[i].getPrice();
            }
            cout<<"Total price: " << totalPrice <<endl;
            return totalPrice;
        }

};

class DiscountService {
    public:
        double applyDiscount(double totalPrice) {
            double discount = 0.0;
            if(totalPrice > 1000.0) {
                discount = totalPrice * 0.1; // 10% discount
            }
            cout<<"Discount applied: " << discount <<endl;
            return discount;
        }
};

//Abstracting payment processing using an interface to adhere to Dependency Inversion Principle and Open/Closed Principle
class PaymentService {
    public:
        //for cash payment amount should be less than 500 
        //with this explicit constraint LSP is also followed as CashPaymentService can be substituted for PaymentService without affecting the correctness of the program
        virtual bool canPay(double amount) {
            return true;
        }

        virtual void processPayment(double amount) = 0;
};


class CardPaymentService : public PaymentService {
    public:
        void processPayment(double amount) {
            cout<<"Processing card payment...Rs" <<amount<<endl;
        }
};

class UPIPaymentService : public PaymentService {
    public:
        void processPayment(double amount) {
            cout<<"Processing UPI payment...Rs" <<amount<<endl;
        }
};

//for cash payment amount should be less than 500
class CashPaymentService : public PaymentService {
    public:
        //for cash payment amount should be less than 500 to adhere to Liskov Substitution Principle
        bool canPay(double amount) override {
            return amount <= 500.0;
        }   

        void processPayment(double amount) {
            if(amount > 500.0) {
                throw invalid_argument("Cash payment amount should be less than Rs 500");
            }
            cout<<"Processing cash payment...Rs" <<amount<<endl;
        }
};

//Deciding payment method at runtime to adhere to Open/Closed Principle
class PaymentProcessor {
    public:
        static PaymentService  *getPaymentService(string mode) {
            if(mode == "CARD") {
                return new CardPaymentService();
            } else if(mode == "UPI") {
                return new UPIPaymentService();
            }
            else if(mode == "CASH") {
                return new CashPaymentService();
            }

            return nullptr;
        }
};

//Abstracting notification services using interfaces to adhere to Interface Segregation Principle, Dependency Inversion Principle, and Open/Closed Principle
class ISMSNotificationService {
    public:
        virtual void send() = 0;
};

class IEmailNotificationService {
    public:
        virtual void send() = 0;
};

class IPushNotificationService {
    public:
        virtual void send() = 0;
};

class SMSNotificationService : public ISMSNotificationService {
    public:
        void send() override {
            cout<<"Sending SMS notification..." <<endl;
        }
};

class EmailNotificationService : public IEmailNotificationService {
    public:
        void send() override {
            cout<<"Sending Email notification..." <<endl;
        }
};

class PushNotificationService : public IPushNotificationService {
    public:
        void send() override {
            cout<<"Sending Push notification..." <<endl;
        }
};

class AllNotificationService : public ISMSNotificationService, public IEmailNotificationService, public IPushNotificationService {
    public:
        void send() override {
            cout<<"Sending SMS notification..." <<endl;
            cout<<"Sending Email notification..." <<endl;
            cout<<"Sending Push notification..." <<endl;
        }
};

class FoodOrderService {   
    private:
        ItemSelector itemSelector;
        PriceCalculator priceCalculator;
        DiscountService discountService;
        PaymentService* paymentService;
        ISMSNotificationService* smsNotificationService = new SMSNotificationService();
        IEmailNotificationService* emailNotificationService = new EmailNotificationService();
        IPushNotificationService* pushNotificationService = new PushNotificationService();

        vector<CartItem> items;

    public:
        FoodOrderService() {
            this->paymentService = nullptr;
        }

        FoodOrderService(PaymentService* paymentService) {
            this->paymentService = paymentService;
        }

        ~FoodOrderService() {
            if (this->paymentService != nullptr) {
                delete this->paymentService;
            }
            if (this->smsNotificationService != nullptr) {
                delete this->smsNotificationService;
            }
            if (this->emailNotificationService != nullptr) {
                delete this->emailNotificationService;
            }
            if (this->pushNotificationService != nullptr) {
                delete this->pushNotificationService;
            }
        }

        void placeOrder() {
            //Select items              
            items = itemSelector.selectItems();

            //Calculate total price
            double totalPrice = priceCalculator.calculatePrice(items);

            //Apply discount
            double discount = discountService.applyDiscount(totalPrice);
            double amount = totalPrice - discount;
            cout<<"Final price: " << amount <<endl;

            if(paymentService->canPay(amount)) {
                //Process payment
                paymentService->processPayment(amount);
            
                //Send notification
                smsNotificationService->send();
                //emailNotificationService->send();
                //pushNotificationService->send();
            } else {
                cout<<"Payment failed. Please choose another payment method." <<endl;
            }

        }
};

int main() {
    PaymentService* paymentService = NULL;
    string mode = "CARD";

    paymentService = PaymentProcessor::getPaymentService(mode);

    FoodOrderService foodOrderService(paymentService);
    foodOrderService.placeOrder();
    
    return 0;
}
```
---
