## Part 1: Test case creation

Following, there is the test we propose to developers. As the QA member of the team you have to design the right and precise test cases to give the team the confidence of a good test coverage. There are no limits nor restrictions, just express yourself.

### Cashier System

Simple cashier function that adds products to a cart and displays the total price. The following products are registered:

| Product Code | Name         | Price  |
|--------------|--------------|--------|
| GR1          | Green Tea    | £3.11  |
| SR1          | Strawberries | £5.00  |
| CF1          | Coffee       | £11.23 |

**Special Rules**

- Buy one get one free
- Buy > N products, pay X price per product
- Buy > N products, pay X% of the original price

**Products and Discount Rules**  
The project doesn't connect to a database, it reads both the products and rules from a YAML file.  
The default location of the file is `priv/assets/products.yml` and `priv/assets/rules.yml`, this can be changed in the Configuration.  
Currently there are only 3 types of configurable discount rules:

- FreeRule (buy N get N free)
- ReducedPriceRule (buy more than N pay a different price)
- FractionPriceRule (buy more than N, pay a percentage of the original price)

---

I’m going to create the testCases from the general to the specifics.  

I’m going to assume that the next 4 points were already tested and their functionalities work properly. Therefore I would only be testing the new function:

1. Products Exist and appear on the screen  
2. The cart exists and appears  
3. Buttons on the shopping display are usable  
4. Buttons on the cart are usable  

---

### Base Cashier System Test Cases

**5 - You can add a product to the cart and the product appears on the cart with the corresponding price**  
When you add a product to the cart the price on the cart changes to the correct amount  
[This tests might depend on if the user is logged in or not]

- **5.1 - Green Tea Test**  
Given a User on the shopping page  
When Clicks on the AddItemButton on GreenTeaItem  
Then GreenTeaItem is added to ShoppingCart  
And ShoppingCartPrice is 3.11

- **5.2 - Strawberries Test**  
Given A User on the shopping page  
When Clicks on the AddItemButton on StrawberriesItem  
Then StrawberriesItem is added to ShoppingCart  
And ShoppingCartPrice is 5

- **5.3 - Coffee Test**  
Given A User on the shopping page  
When Clicks on the AddItemButton on CoffeeItem  
Then CoffeeItem is added to ShoppingCart  
And ShoppingCartPrice is 11.23

*[It might be necessary to test for low connectivity environments and issues related to those, but I’ll focus on functionality]*

**6 - You can delete a product from the cart**  
When you delete the product from the cart the prince on the cart changes to the right amount

- **6.1 - Item Deletion Test**  
Given A User on the shopping page  
And (x)Item on ShoppingCart  
And (x)Item Price is added into ShoppingCartTotal  
When Clicked on DeleteItemButton  
Then (x)Item gets deleted from ShoppingCart  
And (x)ItemPrice gets removed from ShoppingCartTotal  

*We should test this with all the items on the List*  
*It might be necessary to test for low connectivity environments and issues related to those, but I’ll focus on functionality*

---

### Test Case Table

| Test Case ID | Title                     | Steps                         | Expected Result                            |
|--------------|---------------------------|-------------------------------|---------------------------------------------|
| TC-001       | Add Green Tea to cart     | Add 1x Green Tea to cart      | Cart contains 1x Green Tea, total = £3.11   |
| TC-002       | Add Strawberries to cart  | Add 1x Strawberries to cart   | Cart contains 1x Strawberries, total = £5.00|
| TC-003       | Add Coffee to cart        | Add 1x Coffee to cart         | Cart contains 1x Coffee, total = £11.23     |
| TC-004       | Delete item from cart     | Add 1x Green Tea -> Remove it  | Cart is empty, total = £0.00                |
| TC-005       | Delete item from cart     | Add 1x Strawberries -> Remove it | Cart is empty, total = £0.00             |
| TC-006       | Delete item from cart     | Add 1x Coffee -> Remove it     | Cart is empty, total = £0.00                |
| TC-007       | Multiple items + total calc | Add 1x Green Tea + 1x Strawberries + 1x Coffee | Total = £19.34 |

---

### Special Rules Test Cases

These are the assumptions I’m making for the Discount Rules:

**FreeRule**  
When buying 1 Coffee, you get another one free - The discount applies automatically, discounted items don't count towards the discount item needed counter.  
This could be interpreted 2 ways:  
- when adding 1 coffee another one gets added automatically as a “free gift”.  
- or, When 2 items are added, the second one becomes free.  
**I’m going to assume it's the second one.**

#### 7 - Free Rule

- **7.1 - You add 1 coffee**  
The Free Rule doesn't apply  
The price on the cart accounts for one coffee: £11.23

- **7.2 - You add 2 coffee**  
The Free Rule applies so 1 is free  
Price = £11.23

- **7.3 - You add 3 coffee**  
Free Rule applies to 1  
Price = £22.46

- **7.4 - You add 4 coffee**  
2 free -> Pay for 2 -> Total = £22.46  

*Test for 100 products: Pay for 50 -> £561.50. Also test: 5, 10, 15, 20…*

#### Test Cases

| Test Case ID | Title                      | Steps            | Expected Result                     |
|--------------|----------------------------|------------------|-------------------------------------|
| TC-008       | Add 1 Coffee               | Add 1x Coffee    | Total = £11.23                      |
| TC-009       | Add 2 Coffees              | Add 2x Coffee    | 1 Coffee free -> Total = £11.23     |
| TC-010       | Add 3 Coffees              | Add 3x Coffee    | 1 free -> Pay for 2 -> Total = £22.46|
| TC-011       | Add 4 Coffees              | Add 4x Coffee    | 2 free -> Pay for 2 -> Total = £22.46|
| TC-012       | Add 5 Coffees              | Add 5x Coffee    | 2 free -> Pay for 3 -> Total = £33.69|
| TC-013       | Add 100 Coffees            | Add 100x Coffee  | 50 free -> Total = £561.50          |
| TC-014       | Discount removed on removal| Add 3x Coffee -> Remove 1 | Total = £11.23           |

---

**ReducedPriceRule**  
When buying 20 of the items [Strawberries], the price gets changed accordingly [£5 -> £3].

#### 8. Reduced Price Rule

- **8.1** Add 1 -> No discount  
- **8.2** Add 19 -> No discount  
- **8.3** Add 20 -> £60 (3x20)  
- **8.4** Add 21 -> £63 (3x21)

#### Test Cases

| Test Case ID | Title                             | Steps                     | Expected Result                     |
|--------------|-----------------------------------|---------------------------|-------------------------------------|
| TC-015       | Add <20 Strawberries              | Add 19x Strawberries      | Price = £95.00                      |
| TC-016       | Add 20 Strawberries               | Add 20x Strawberries      | Discount: £3 -> Price = £60.00      |
| TC-017       | Add 25 Strawberries               | Add 25x Strawberries      | Discount: £3 -> Price = £75.00      |
| TC-018       | Discount removed on quantity drop | Add 20 -> Remove 1         | Price = £95.00 (no discount)       |

---

**FractionPriceRule**  
When item quantity is equal or over 4, get a 50% discount - applies to Green Tea.

#### 9. FractionPriceRule

- **9.1** Add 1 Green Tea -> £3.11  
- **9.2** Add 3 Green Tea -> £9.33  
- **9.3** Add 4 Green Tea -> £6.22 (discounted)

#### Test Cases

| Test Case ID | Title             | Steps             | Expected Result        |
|--------------|-------------------|-------------------|------------------------|
| TC-019       | Add 3 Green Teas  | Add 3x Green Tea  | No discount -> £9.33    |
| TC-020       | Add 4 Green Teas  | Add 4x Green Tea  | 50% off -> £6.22        |
| TC-021       | Add 10 Green Teas | Add 10x Green Tea | 50% off -> £15.55       |
| TC-022       | Discount removed on quantity drop  | Add 4x Green Tea -> Remove 1  | 50% off -> £6.22 -> No discount -> £9.33       |

---

### Scenarios Combined

Once we know that the functionalities behave properly by themselves, we would need to test to see if they work properly with each other.

| Test ID | Products Added                       | Action          | Expected Discount(s)                        | Expected Result                         |
|---------|--------------------------------------|-----------------|---------------------------------------------|------------------------------------------|
| TC-023  | 4 Green Teas, 2 Coffees              | Add             | Green Tea (50% off), Coffee (2-for-1)       | Green Tea at half price, Coffee 2-for-1  |
| TC-024  | 4 Green Teas, 20 Strawberries        | Add             | Green Tea (50%), Strawberries (£3 each)     | Both discounts apply                     |
| TC-025  | 4 Green Teas, 20 Strawberries, 2 Coffees | Add         | All three discounts                         | All correctly applied                    |
| TC-026  | 20 Strawberries, 2 Coffees           | Add             | Strawberries + Coffee                       | Discounts apply without Green Tea        |
| TC-027  | 4 Green Teas, 2 Coffees -> remove Teas| Remove Teas     | Coffee stays discounted                     | Coffee keeps discount                    |
| TC-028  | 4 Green Teas, 20 Strawberries -> remove Teas | Remove Teas | Strawberries stay discounted                | Strawberries discount persists           |
| TC-029  | 4 Green Teas, 20 Strawberries -> remove Strawberries | Remove Strawberries | Green Tea discount persists      | Green Tea discount persists              |
| TC-030  | All -> remove Coffees                 | Remove Coffees  | Green Tea + Strawberries discounts stay     | Tea & Strawberries stay discounted       |
| TC-031  | 20 Strawberries -> remove 1           | Reduce quantity | None                                        | Discount removed, price = £95            |
