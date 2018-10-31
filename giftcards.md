# Gift Cards

Application of gift card payments at checkout

## Current Mechanism

Gift card input is at payment step of checkout

1. Customer enters Vii number + Vii pin, hits Apply

2. The follow info is sent to CheckBalance endpoint on SHIP-Order

    ```js
    {
      viiCardNumber,
      viiCardPin,
      cartTotal
    }
    ```

3. SHIP-Order will run a check balance of the Vii gift card with the CardNumber and PIN

4. If Vii returns OK with the balance left on the Vii card, SHIP-Order will call out to Shopify's gift card API and generate a giftcard with a **randomised code** of balance depending on whether the Vii balance is less than the cart total (eg. 9h7f74e4ghf)

5. SHIP-Order will then return to the client the following:

    ```js
    AmountApplied,
    ShopifyGiftCardCode,
    shopifyGiftCardStatus,
    ViiGiftCardNumber,
    viiBalance
    ```

6. On receipt of the SHIP-Order return, the last 4 digits of ShopifyGiftCardCode are sliced to be used as the key for LocalStorage
    1. These details are JSON stringified to store the following values in LS:
        ```js
        4ghf:
        {
            AmountApplied,
            AvailableBalance,
            CartTotal,
            Status (not so important),
            ViiGiftCardNumber
        }
        ```
    2. After values are stored, the ShopifyGiftCardCode is entered into a hidden shopify giftcard field in the DOM, and a form submit is triggered
    3. Page will refresh
    4. The page will load with the shopify giftcard set in the sidebar that shows the last 4 digits of the shopify giftcard code and the amount of the giftcard applied (varies automatically with the cart)
    5. We use these last 4 digits to query LocalStorage for the applied gift card details and display to customer
        - ViiGiftCardNumber
        - Amount Remaining (AvailableBalance - AmountApplied)
        - AmountApplied

7. Any gift card(s) applied are cached by Shopify into the checkout (we think it's specific to the checkout cookie)
    - Expires within 30 days ?

8. Customer can then go on to complete payment with gift cards covering the whole order, or input credit card for split payment

## Issues

While the basic flow works, there are several issues affecting gift cards...

1. Cart changes
    - Coupons
    - Change in cart contents
    - Current implementation is 1-pass only, any changes to cart total will render the gift card stale
    - Current work-around is to detect cart total change and clear the gift cards (jQuery)

2. Hidden at checkout steps
    - Gift cards hidden at checkout steps (customer information & shipping)
    - Cart Total presented to customer modified (knock-on effects)

3. Gift Card expenditure outside of Shopify
    - Vii gift card usage won't be reflected in Shopify (Shopify gift card retains same value)

4. ???
    - Usage alongside Paypal?