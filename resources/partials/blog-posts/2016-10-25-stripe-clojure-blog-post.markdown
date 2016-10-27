
# Getting Started with Stripe-Clojure

## Introduction

Stripe is a leading provider of payment and subscription management solutions for many businesses. The most notable feature of Stripe is their nearly immaculate implementation of a REST API. Developers can simply integrate the API or build applications to automate away many business pains such as routinely charging customers or editing/adding payment information.

Stripe-Clojure is a Clojure development library which is streamlined for easy functional implementation of the Stripe API. The library handles API calls to Stripe which allows developers to easily interact with the API in a clean, easy to use functional environment. This article will cover the basics of the Stripe-Clojure library along with a few examples to get started accepting payments. The tutorial will be covered using emacs with a cider REPL (EPX Labs will soon be open sourcing our emacs-live + add-ons development environment affectionately named the "labkit").

Stripe-clojure also easily connects with Amazon Web Services as part of a serverless system which has greater reliability and costs less than running traditional servers. [EVAN expand serverless benefits section.]

## Stripe API Basics

### The Customer Object

The customer object is the core of every operation available through Stripe. Customer objects contain all of the information relevant to a customer and are required for charges and subscriptions. Each customer has a unique ID which is used for payments or to start a subscription. The Stripe-clojure library uses a functional implementation of the customer object.

### Subscriptions

Subscriptions are used to bill customers a set price every month. A Stripe account can have many different plans available and customers may have one or many different subscriptions. 

### Sources

Customer payment information is stored securely in Stripe's database as a source. When creating a source for a customer it is not necessary to hold any payment details which removes a potential security risk. It is also a good practice to communicate with Stripe using tokenized information. 

## Getting started with the library

Today we will show the basic process of creating a test customer using the Stripe-Clojure library. We will also run through the process of creating a subscription and processing payments. When testing the library it is important to use the "test" key so that no real payments are made. 

### Preparing to communicate with Stripe API

In order to make changes to your Stripe account using the API you must have permission. Stripe has different keys that entitle different permissions. Your API keys can be found on the Stripe dashboard under account settings. For this example we will be using the "private test key".

Once you find your API key, boot up your environment and load up Emacs. You can then open the Stripe-clojure core.clj file and additionally open a REPL. 
After the repl has been loaded, switch into the stripe-clojure namespace. The API key can be added simply by using the `set-tokens!`  command with a map of your secret test key as the parameter.

```clojure
(set-tokens! {:private "sk_test_xxxxxxxxxxxxxxxx"})
```

### Adding a customer to Stripe

The stripe-clojure library makes it simple to add a customer to Stripe. Customers can be created with many different types of data however we will be initializing our test customer with only an email. The customer is created using a POST request which can be done through the `create` command. The `create` command accepts a map with a route and also a map of parameters related to the object. In this case our request will look like:

```clojure
(create {:customers {:email "stripe-test-man@test.com"}})
```

If the previous step was done correctly, Stripe should return a customer map:

```clojure
{:default_source nil,
 :description nil,
 :email "stripe-test-man@test.com",
 :delinquent false,
 :account_balance 0,
 :sources
 {:object "list",
  :data [],
  :has_more false,
  :total_count 0,
  :url "/v1/customers/cus_9RQwyEK0fHnSkO/sources"},
 :created 1477414020,
 :subscriptions
 {:object "list",
  :data [],
  :has_more false,
  :total_count 0,
  :url "/v1/customers/cus_9RQwyEK0fHnSkO/subscriptions"},
 :currency nil,
 :id "cus_9RQwyEK0fHnSkO",
 :livemode false,
 :shipping nil,
 :discount nil,
 :metadata {},
 :object "customer"}
```

In other cases, different parameters can be used with the parameter map. For example a default source could be added on customer creation but that will be done in the next step.

### Adding a default payment source

Payment sources can be added using either a map of credit card details or a tokenized form of the details. In this example we will create a test token using Stripe's token generator. A token simply represents a single credit card and can only be used on one customer. In order to add the token to the customer, we can use the same command as we did previously. In the case the route will be the same however we will just include a different parameter `:source`. We also need to include the Stripe customer ID of the customer we want to add the source to. 

```clojure
(create {:customers {:customer_id "cus_9RQwyEK0fHnSkO" :source "tok_198YCVJ3Zx6FNnHuZAMlkkX6"}})
```

### Starting a Subscriptions
The previous two steps were necessary to begin charging customers on a monthly basis. This step requires a plan to exist on the linked stripe account. The key points of stripe plans are the price and the billing frequency. For our purposes we are trying to subscribe a customer to a plan named "stripe-clojure" that charges $9.99 and bills monthly. 
In the repl we can run a similar command to what we have been using to edit customer data. 

```clojure
(create {:customers {:customer_id "cus_9RQwyEK0fHnSkO" :plan "stripe-clojure"}})
```
This returns a success response with the updated customer object to show that the request worked.

```clojure
{:default_source "card_198YCVJ3Zx6FNnHuyeItfoJu",
 :description nil,
 :email "stripe-test-man@test.com",
 :delinquent false,
 :account_balance 0,
 :sources
 {:object "list",
  :data
  [{:address_line1_check nil,
    :address_state nil,
    :dynamic_last4 nil,
    :address_zip_check nil,
    :tokenization_method nil,
    :exp_year 2017,
    :name nil,
    :cvc_check "pass",
    :last4 "4242",
    :brand "Visa",
    :customer "cus_9RQwyEK0fHnSkO",
    :address_country nil,
    :funding "credit",
    :address_line2 nil,
    :id "card_198YCVJ3Zx6FNnHuyeItfoJu",
    :address_zip nil,
    :address_line1 nil,
    :exp_month 12,
    :country "US",
    :metadata {},
    :object "card",
    :fingerprint "mRNahc5gc8EgfkI5",
    :address_city nil}],
  :has_more false,
  :total_count 1,
  :url "/v1/customers/cus_9RQwyEK0fHnSkO/sources"},
 :created 1477414020,
 :subscriptions
 {:object "list",
  :data
  [{:canceled_at nil,
    :application_fee_percent nil,
    :start 1477453490,
    :created 1477453490,
    :current_period_end 1480131890,
    :trial_end nil,
    :customer "cus_9RQwyEK0fHnSkO",
    :ended_at nil,
    :status "active",
    :id "sub_9RbY6wnlhYeBdZ",
    :cancel_at_period_end false,
    :livemode false,
    :quantity 1,
    :trial_start nil,
    :discount nil,
    :plan
    {:amount 999,
     :name "stripe-clojure",
     :created 1477453200,
     :currency "usd",
     :interval_count 1,
     :id "stripe-clojure",
     :trial_period_days nil,
     :interval "month",
     :livemode false,
     :metadata {},
     :object "plan",
     :statement_descriptor nil},
    :current_period_start 1477453490,
    :metadata {},
    :object "subscription",
    :tax_percent nil}],
  :has_more false,
  :total_count 1,
  :url "/v1/customers/cus_9RQwyEK0fHnSkO/subscriptions"},
 :currency "usd",
 :id "cus_9RQwyEK0fHnSkO",
 :livemode false,
 :shipping nil,
 :discount nil,
 :metadata {},
 :object "customer"}
```
### Taking it a step further

The Stripe-Clojure library provides the basis for a functional environment with stripe capability. It contains all of the tools needed to build complex payment and customer management systems and can be used in conjunction with AWS to provide a serverless solution to payment processing. If you would like to know more about going serverless, contact EPXlabs to find out how you could improve server reliability while cutting costs. 