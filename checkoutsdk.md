# Creating a custom checkout using the BigCommerce Checkout SDK

## What is the BigCommerce SDK?

The BigCommerce Checkout SDK allows for custom design to the Unified Checkout Out. By allowing BigCommerce to handle the functionality, designers and developers can
focus on creating a customized checkout experience. 
This tutorial will walk through creating a checkout using React.

Want to jump to a the finished project? The Checkout SDK sample app can be downloaded (here)[add-link-here]. Needs link

## First Steps:

This app is written in React (JSX) and is made of components that can be used together in designing the checkout. 
The components are cart, customer, shipping and billing.

Components we are using:(Should these have definitions?)

Cart [Link each one to the final object]
Customer
Shipping
Billing


## Quick Notes: 

* This tutorial assume knowledge of JavaScript, HTML, CSS and [React](https://reactjs.org/). 
* BigCommerce stores using Stencil must also have Optimized [One Page Checkout](https://support.bigcommerce.com/articles/Public/Optimized-Single-Page-Checkout/) enabled otherwise the changes will not be read when Stencil is started.
* This also works on legacy Blueprint Stores. 
* To troubleshoot React issues it helps to use the [React Chrome plugin](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi) from Facebook. It creates a new tab in the devloper console where React specific errors are located. 
* You will need to either make a copy of your existing Stencil theme or download a copy of the [Cornerstone theme](https://github.com/bigcommerce/cornerstone.git) and be familiar with the [Stencil CLI](https://stencil.bigcommerce.com/). The Stencil CLI and Cornerstone dependencies will need to be installed before hand.
* Having a few items added to your cart locally will help in following along. 
* There should be an item with a gift certificate and an item with a coupon code added since they will be used in the example app later.
* Knowing how to use the developer tools console will give you access to the object which will be needed to see how objects should be referenced.

## Add Checkout Page Object

In the app.js we need to add the theme reference for checkout.
Then in checkout.html we are adding 3 scripts. 

`<script>window.__webpack_public_path__ = "{{cdn 'assets/dist/'}}";</script>` This is allowing us to use webpack to specify the base path for all assets. `window` is an object supported by all browsers. It references the browsers window. 

`<script src="{{cdn 'assets/dist/theme-bundle.main.js'}}"></script>` This is where the main theme bundle is located.

```
<script>
    window.stencilBootstrap("{{page_type}}", {{jsContext}}).load();
</script>
```
This is loading Bootstrap into out stencil theme. 


Run `stencil start` and navigate to your localhost/checkout and open the console. 
Should see the Checkout page in the console. [add screenshot here]

Files changed:
```
assets/js/app.js
assets/js/theme/checkout.js
templates/pages/checkout.html
```

## Import React

We are going to remove the checkout.js. We know the checkout location is working after being added to the app.js after rendering it in the console. 
In `package.json` we need to add `"react": "^16.0.0"` and `"react-dom": "^16.0.0"`. We also need to add 
`"babel-preset-react": "^6.24.1"`. [Babel](https://babeljs.io/) is a complier that takes JSX and converts it into JavaScript for the browser. The Babel plugin is needed to support using JSX within React. 

Now we need to update `webpack.conf.js`. React uses the file extension .jsx instead of .js. JSX is an extension to JavaScript and create React elements. JSX needs to be added as part of the extensions.
Run `npm install` here since we added to the package.json file. 

Add a new folder `checkout` to `assets/js/theme/` and in that folder add a new file `checkout.jsx`. The final path should look like `assets/js/theme/checkout/checkout.jsx`. Here we import React and create the `CheckoutComponent` for the app. Right now its going to load the word Checkout on the page. 

Add another `checkout.jsx` file in the main `theme` folder. The final path should be `assets/js/theme/checkout.jsx`
This page imports our `CheckoutComponent` and renders it in the DOM. `<Checkout />` is our app which is going to be rendered in the `checkout-app` div we are going to create. 

In `checkout.html` replace `{{{ checkout.checkout_content }}}` with `<div id="checkout-app"></div>`

At this point you should see Checkout in the React Console and the checkout page. (add the screenshot here)

### Files Changed:

```
assets/js/theme/checkout.js
assets/js/theme/checkout.jsx
assets/js/theme/checkout/checkout.jsx
package.json
templates/pages/checkout.html
webpack.conf.js
```

## Import SDK

Now that React has been imported and we setup the skeleton for the checkout app, we are going to import the SDK. In your `package.json` add 
`@bigcommerce/checkout-sdk": "git+ssh://git@github.com:bigcommerce/checkout-sdk-js.git"`.
Run `npm install` to install the package.

In `assets/js/theme/checkout/checkout.jsx` we need to initialize a `CheckoutService`. Then we are going to add in a Checkout. In this code we are importing React and the CheckoutService[link to this object]. The CheckoutService loads the intial checkout state. We first call `construtor(props)` then `super(props)` since we are accessing the `props` inside the construtor class we need to use `super(props)`. To learn more about [`constructor`](https://reactjs.org/docs/react-component.html#constructor). After the component is mounted we log to the console the cart created, which has moved into the checkout portion. 

```
import React from 'react';
import { createCheckoutService } from '@bigcommerce/checkout-sdk';

export default class CheckoutComponent extends React.Component {
    constructor(props) {
        super(props);

        this.service = createCheckoutService();
    }

    componentDidMount() {
        this.service.loadCheckout()
            .then(({ checkout }) => console.log(checkout.getCart()));
    }

    render() {
        return (
            <section>
                Checkout
            </section>
        );
    }
}

```

Run `stencil start` and go to `http://localhost:3000/checkout` and open the the console to see a checkout object. Add something to the cart and then navigate back to the checkout to see more information loaded in the object. Keep this object in mind since it will serve as a reference for what you can show on the checkout and is needed in the next step.

[add image here of console with object] - also link to a gist where they can review it there


Files Changed:

```
assets/js/theme/checkout/checkout.jsx
package.json
```


## Add Cart

We are going to add `accounting` which helps with number and currency formatting and `material-ui` which imports Google's material UI. The material ui is an optional step and you can use anything you like for styling.
Run `npm install` and restart stencil cli if needed. Update `assets/js/theme/checkout/checkout.jsx` and create `assets/js/theme/checkout/cart.jsx` to match the the sample app. Refresh checkout and now there should be Cart with the items if you have added any, Subtotal, Shipping, Tax and Total. 
[add image]

Let's go over `assets/js/theme/checkout/cart.jsx`. This file contains the Items,Subtotal, Shipping, Tax and Grand Total. It is broken down into list items, that can be removed or changed to create a customized option. 

This section loops through the items tha that have been added to the cart and displays, the name, picture(s), amount and quantity.

```
...

{ (this.props.cart.items || []).map((item) => (
                        <ListItem key={ item.id }>
                            <Avatar src={ item.imageUrl } />
                            <ListItemText primary={ `${item.quantity} x ${item.name}` } />
                            <ListItemSecondaryAction>
                                { formatMoney(item.amountAfterDiscount) }
                            </ListItemSecondaryAction>
                        </ListItem>
                    )) }

...

```

For example the current checkout page does not list Coupons or Gift Certificates. To add them copy and paste one of the lists. Then update the `ListItemText` and the `ListItemSecondaryAction`. 

```
                    <ListItem>
                        <ListItemText primary="Gift Certficate" />
                        <ListItemSecondaryAction>
                            { formatMoney(this.props.cart.giftCertificate.amount) }
                        </ListItemSecondaryAction>
                    </ListItem>
```                    

Stencil should have reloaded at this point and now you can see a new line that says Gift Certificate, but it shows $0. If you have a gift certificate added to an item in your cart, you can see that $0 is incorrect. We need to see what to call for the Gift Certificate to get the amount. In the 
`assets/js/theme/checkout/checkout.jsx` add `console.log(checkout.getCart());`. You can add it right under `const { checkout } = this.service.getState();`. Open the dev tool and the object should be printed to the console. Find `giftCertificate` and then `totalDiscountAmount`. Update the `ListItem` to read: 
```
                    <ListItem>
                        <ListItemText primary="Gift Certficate" />
                        <ListItemSecondaryAction>
                            { formatMoney(this.props.cart.giftCertificate.totalDiscountedAmount) }
                        </ListItemSecondaryAction>
                    </ListItem>

```

Make sure the page refreshes and now the Gift Certificate Amount is displayed. To add the coupon `this.props.cart.coupon.discountedAmount`. 

In `assets/js/theme/checkout/checkout.jsx` there were are a few changes that need to be made. This `.then(({ checkout }) => console.log(checkout.getCart()));` was replaced with `.then(() => this.setState({ isLoading: false }));` [`setState`](https://reactjs.org/docs/react-component.html#setstate) queues changes to the component and tell React the componenet and its children need to updated(re-rendered) with the new state. 

in `assets/js/theme/checkout/checkout.jsx`, the text Checkout needed to be replaced by the Cart portion of the checkout. 

```
        return (
            <section>
                <Cart cart={ checkout.getCart() } />
            </section>
        );
    }

```

Files Changed:

```
assets/js/theme/checkout/cart.jsx
assets/js/theme/checkout/checkout.jsx
package.json
````

## Add Customer (Anything else I should note here...?)

Create file `customer.jsx` in `assets/js/theme/checkout/`. It should be `assets/js/theme/checkout/customer.jsx` This where we are going to add in the code to pull the information about which customer the checkout belongs to. Add the code from [add link here]

After the page reloads, there is a section for customers to login. After loggin in it shows a simple message confirming the customer has logged in.[add - image here].

In React you can also use setState when logging customers in and out. `componentWillReceiveProps` is when  `props` are passed to the Component instance. This is useful for when a customer needs to type into a input or this case a email and password box. It will compare the incoming proprs to the current props and depending on the value React can make a decision on what to do next. 
Open the React console and start typing into the email and password box. You can see the values being updated. 
[add gif here]

The inital state is a blank email address and password. The component should expect values from those props and they are waiting on change. 
The state change happens onSubmit when the button is clicked and when there is data in the email/password. Clicking signin with nothing there doesn't generate any feedback. 

The `&&` in the `this.props.customer.isGuest` is saying if the customer is a guest & this other data/state exists. 

Now we are going to update the `assets/js/theme/checkout/checkout.jsx` to render the customer section that was created. We need to update the
`const` to handle errors and add in the customer. 

(Need to explain this more)


Files Changed:

```
assets/js/theme/checkout/checkout.jsx
assets/js/theme/checkout/customer.jsx
```

## Add Shipping
Since React allows for components to be split for use later, shipping will be broken into shipping and shipping-options. Where `Shipping` will contain the `Shipping Options` component. 

The `Shipping Options` component will contain what is setup on the store for customers to get items shipped. In the example image Flat Rate is enabled on the store. 

Create the file `assets/js/theme/checkout/shipping-options.jsx`. It will contain the ShippingOptionsComponent

[Add Image here of Shipping console]


Files Changed:

```
assets/js/theme/checkout/address.jsx
assets/js/theme/checkout/checkout.jsx
assets/js/theme/checkout/shipping-options.jsx
assets/js/theme/checkout/shipping.jsx
```

## Add Billing

## Final Steps

## Troubleshooting
* Did you remember to run `npm install` after changing the `package.json`?
* If using a new copy of Cornerstone, did you follow all the steps to get [Stencil CLI](https://stencil.bigcommerce.com/docs/installing-and-launching-stencil-1) installed? 
* Make sure for Stencil you are using NPM versions between 4x - 7x.
* Check the React tab in the console to step through the app
* In this React example the only thing `assets/js/checkout.jsx` is going is rendering the  `Checkout` created to the DOM. All edits should be in the `checkout/... ` folder after the initial setup. 

