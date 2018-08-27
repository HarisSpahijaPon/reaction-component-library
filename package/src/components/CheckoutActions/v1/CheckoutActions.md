### Overview
The `CheckoutActions` component is responsible for:
  * Displaying each `CheckoutAction` and managing it's status. (complete, incomplete, active)
  * Providing each  `CheckoutAction` with any data it may need from the `cart` object.
  * Capturing/Editing of `CheckoutAction` data.
  * Rendering captured `CheckoutAction` data in a `CheckoutActionComplete` component.

#### Usage
`CheckoutActions` takes an array of `actions`, each `action` needs a `label` and a checkout action component that will be responsible to capturing a piece of checkout data as well as a `props` object to include the piece of `cart.checkout` data the action needs to display and a `onSubmit` function that needs to be called during action submission.

**Example of Actions array**
```js static
const actions = [
  {
    label: "Shipping Information",
    status: "incomplete",
    component: ShippingAddressCheckoutAction,
    onSubmit: setShippingAddress
    props: {
      fulfillmentGroup: cart.checkout.fulfillmentGroup
    }
  },
  {
    label: "Shipping Options",
    component: ShippingOptionCheckoutAction,
    onSubmit: setShippingOption
    props: {
      availableFulfillmentGroups : cart.checkout.fulfillmentGroup.availableFulfillmentGroups
    }
  },
  { 
    label: "Payment Information", 
    status: "incomplete",
    component: PaymentCheckoutAction,
    onSubmit: setPayment,
    props: {
      payment: cart.checkout.payments[0]
    }
  }
];

```

**Note:** These examples only use the `ShippingAddressCheckoutAction` as the actions component. This will be updated with more actions as they get created.

```jsx
const fulfillmentGroups = [{
  _id: 1,
  type: "shipping",
  data: {
    shippingAddress: null
  }
}];

const paymentMethods = [{
  _id: 1,
  name: "reactionstripe",
  data: {
    billingAddress: null,
    displayName: null
  }
}];

class CheckoutActionsExample extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      checkout: {
        fulfillmentGroups,
        payments: paymentMethods
      }
    }

    this.setShippingAddress = this.setShippingAddress.bind(this);
    this.setPaymentMethod = this.setPaymentMethod.bind(this);
  }

  getShippingStatus() {
    const groupWithoutAddress = this.state.checkout.fulfillmentGroups.find((group) => {
      const shippingGroup = group.type === "shipping";
      return shippingGroup && !group.data.shippingAddress;
    });

    return (groupWithoutAddress) ? "incomplete" : "complete";
  }

  getPaymentStatus() {
    const paymentWithoutData = this.state.checkout.payments.find((payment) => {
      return !payment.data.displayName;
    })

    return (paymentWithoutData) ? "incomplete" : "complete";
  }

  setShippingAddress(data) {
    const { checkout } = this.state;

    return new Promise((resolve, reject) => {
        setTimeout(() => {
          this.setState(Object.assign(this.state, {
            checkout: {
              payments: checkout.payments,
              fulfillmentGroups: [{
                data: {
                  shippingAddress: data 
                }
              }]
            }
          }));
          resolve(data);
        }, 1000, { data });
    });
  }

  setPaymentMethod(data) {
    const { billingAddress, token: { card } } = data;
    const { checkout } = this.state;
    const payment = {
      data: {
        billingAddress,
        displayName: `${card.brand} ending in ${card.last4}`
      }
    }

    return new Promise((resolve, reject) => {
        setTimeout(() => {
          this.setState(Object.assign(this.state, {
            checkout: {
              fulfillmentGroups: checkout.fulfillmentGroups,
              payments: [payment] 
            }
          }));
          resolve(payment);
        }, 1000, { payment });
    });
  }

  render() {
    const { checkout } = this.state;

    const actions = [
      {
        label: "Shipping Information",
        status: this.getShippingStatus(),
        component: ShippingAddressCheckoutAction,
        onSubmit: this.setShippingAddress,
        props:  { 
          fulfillmentGroup: checkout.fulfillmentGroups[0]
        }
      },
      { 
        label: "Payment Information", 
        status: this.getPaymentStatus(),
        component: StripePaymentCheckoutAction,
        onSubmit: this.setPaymentMethod,
        props: {
            payment: checkout.payments[0] 
        } 
      }
    ];

    return (
      <CheckoutActions actions={actions} />
    )
  }
}
;<CheckoutActionsExample />
```