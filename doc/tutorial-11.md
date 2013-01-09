# Tutorial 11 - Handling events using Domina+Hiccups and c2

In this tutorial we get a brief insight in event handling by means of
[domina library][1] and [hiccups][2] or the [c2 library][9]. Actually,
*c2* is a declarative data visualization library which goes far beyond
the usage presented here. We refer to next tutorials or the
[readme][10] for further details on data driven visualization with
*c2*.

## Introduction

Let's go back to the shopping calculator form introduced in
[tutorial 04][3], our aim is to enrich that form in order to provide
some examples of events, handled with [domina library][1]. Following
Larry [Larry Ullman][5] in
[Modern JavaScript: Development and Desing][4], events can be
distinguished in four cathegory: *input device* and *browser*, which
will be further investigated here, and *keyboard* and *form*.

At this point our login form is the following 

![Shopping Page][6]

and it is produced by the following HTML source `shopping-event.html`

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Shopping Calculator</title>
    <!--[if lt IE 9]>
    <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <!-- shopping.html -->
  <form action="" method="post" id="shoppingForm" novalidate>
    <legend> Shopping Calculator</legend>
    <fieldset>
      <div>
        <label for="quantity">Quantity</label>
        <input type="number"
               name="quantity"
               id="quantity"
               value="1"
               min="1" required>
      </div>
      <div>
        <label for="price">Price Per Unit</label>
        <input type="text"
               name="price"
               id="price"
               value="1.00"
               required>
      </div>
      <div>
        <label for="tax">Tax Rate (%)</label>
        <input type="text"
               name="tax"
               id="tax"
               value="0.0"
               required>
      </div>
      <div>
        <label for="discount">Discount</label>
        <input type="text"
               name="discount"
               id="discount"
               value="0.00" required>
      </div>
      <div>
        <label for="total">Total</label>
        <input type="text"
               name="total"
               id="total"
               value="0.00">
      </div>			
	  <br>
	
	
      </br>
      <div>
        <input type="submit"
               value="Calculate"
               id="calculate">
      </div>
      <div>
        <input type="submit"
               value="Reset"
               id="reset">
      </div>
    </fieldset>
  </form>
  <script src="js/modern.js"></script>
  <script>modern_cljs.shopping.init();</script>
</body>
</html>
```


## A mouseover/mouseout event

Together with the calculate and reset feature associated with the
click, we want to add a mouseover/mousout event which print on the
form the behavior of the buttons *Calcuate* and *Reset*, that is

![Shopping events][7]

To do this, we define the functions `addcalclauncher` and
`addresetlauncher`, that handle the events associated with the
calcuate and reset events of *Calcuate* and *Reset* buttons. The
following code handles within a unique function the three
behaviors concerning the *Calcuate* button.

```clj
(defn addcalclauncher []
  (doall
   [(evts/listen! (dom/by-id "calculate") :mouseover (fn [evt] (appendinfo)))
    (evts/listen! (dom/by-id "calculate") :mouseout (fn [evt] (removeinfo)))
    (evts/listen! (dom/by-id "calculate") :click  (fn [evt] (calculate)))]))

(defn ^:export init []
  (if (and js/document
           (.-getElementById js/document))
     (addcalclauncher)))
```

provided that we have included the domina library running

```clj
(ns event-ex-one.reset
	(:require [domina :as dom]
	          [domina.events :as evts]))
```

Here `calculate` is the "calculator" routine specified in the previous
tutorials. The functions `appendinfo`, `removeinfo` are responsible to
the information printing on the form. To manipulate the underlining
HTML we proceed as follows

```cljs
(defn appendinfo []
  (dom/append! (xdom/xpath "//body/form") (hsc/html [:div {:id "txtcalc"} "Click to calculate"])))

(defn removeinfo []
  (dom/destroy! (dom/by-id "txtcalc")))
```

The `dom/append!` and `dom/destroy!` functions respectively add and
delete a specified DOM element. A review of the specifications which
can be passed to these function can be found in the
[domina readme][1]. Since `dom/append!` receives as second argument a
string which contains the HTML code to be appended, we want to have a
more "clojurish" approach to generate HTML code. To this aim, we use
[hiccups library][2]. It provides the function `hsc/html` which
return a string containing an HTML source code defined by a standard
"hiccups" syntax (see [hiccups readme][2] for a complete review).

Similary, we hook to the *Reset* buttons the similar events.

```clj
(defn addresetlauncher []
  (doall
   [(evts/listen! (dom/by-id "reset") :mouseover (fn [evt] (appendinfor)))
    (evts/listen! (dom/by-id "reset") :mouseout (fn [evt] (removeinfor)))
    (evts/listen! (dom/by-id "reset") :click  (fn [evt] (resetform)))]))

;; the same as the previous sample
(defn ^:export init []
  (if (and js/document
           (.-getElementById js/document))
     (addresetlauncher)))
```

Here `reset` is the function that reset the form.

```clj
(defn resetform []
  (dom/set-value! (dom/by-id "total") "0.00")
  (dom/set-value! (dom/by-id "price") "0.00")
  (dom/set-value! (dom/by-id "tax") "0.0")
  (dom/set-value! (dom/by-id "discount") "0.00")
  (dom/set-value! (dom/by-id "quantity") "1")  
  false)
```

As shown in [tutorial 6][8], to make the produced JavaScript actually
runnable, we need to add in the HTML `shopping.html` the following
lines

```HTML
<script>modern_cljs.shopping.init();</script>
<script>modern_cljs.reset.init();</script>
```

> Since the events are hooked both to the DOM element and its
> response, there is no behavior difference between the bubble-phase
> and the capture-phase, anyway domina allows the user follow both the
> approaches.

## Another approach for building the page

We have seen above how [domina][1] and [hiccups][2] can be expolited
for HTML pages manipulations. Anyway a different approach is possible,
that is we can build an html page entirely in the ClojureScript code
and declaring only a minimal skeleton in our HTML. To do so we employ
the [c2 library][9].

The HTML page is now the following.

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Shopping Calculator</title>
    <!--[if lt IE 9]>
<script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
<![endif]-->
<link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <form action="" method="post" id="shoppingForm" novalidate></form>   
  <script src="js/modern_dbg.js"></script>
</body>
</html>
```

and the shopping form can be initialized using the `bind!` macro of
`c2.util`, which must be imported by the usual `use-macro`
declaration.

```clj
(bind! "#shoppingForm"
       [:form
        [:legend "Shopping Calculator"]
        [:fieldset
         [:div [:label {:for "quantity"} "Quantity"
                [:input#quantity {:type "number"
                                  :name "quantity"
                                  :value "1"
                                  :min "1"
                                  :required true}]]]
         [:div [:label {:for "price"} "Price Per Unit"
                [:input#price {:type "text"
                               :name "price"
                               :value "1.00"
                               :required true}]]]
         [:div [:label {:for "tax"} "Tax Rate (%)"
                [:input#tax {:type "text"
                             :name "tax"
                             :value "0.0"
                             :requried true}]]]
         [:div [:label {:for "discount"} "Discount"
                [:input#discount {:type "text"
                                  :name "discount"
                                  :value "0.0"
                                  :required true}]]]
         [:div [:label {:for "total"} "Total"
                [:input#total {:type "text"
                               :name "total"
                               :value "0.00"
                               :required true}]]]
         [:div [:input#calculateButton {:type "button"
                                         :value "Calculate"}]]
         [:div [:input#resetButton {:type "button"
                               :value "Reset"}]]]])

```

> Observe that the basic difference between this approach goes beyond
> the language we use to build a HTML page. Following this approach
> the actions associated with the DOM elements don't require to be
> initialized (see the *init* functions in the previous tutorials) and
> so, no CLJS functions must be exported, as we shall see below.

We see now how the events discussed in the previous section can be
handled with *c2*.

## The mouseover/mouseout event with c2

We recall that we want to attach to the *Calculate* button a mouseover
event which prints on the form some information about the behavior of
the button, which must disappear when the mouse moves out the
button. Similary for the *Reset* button.

Here the code for the calculation

```clj
	(c2event/on-raw "#calculateButton" :click calculate)
	(c2event/on-raw "#calculateButton" :mouseover (fn [] (add-info "#shoppingForm" "calculate")))
	(c2event/on-raw "#calculateButton" :mouseout (fn [] (remove-info "#calculate")))
```

and the code for the reset action

```clj
	(c2event/on-raw "#resetButton" :click reset-form)
	(c2event/on-raw "#resetButton" :mouseover (fn [] (add-info "#shoppingForm" "reset")))
	(c2event/on-raw "#resetButton" :mouseout (fn [] (remove-info "#reset")))
```

where `calculate`, `reset`, `add-info` and `remove-info` are now defined as follows

```clj
	(defn calculate []
		(let [quantity (c2dom/val "#quantity")
			price (c2dom/val "#price")
			tax (c2dom/val "#tax")
			discount (dom/val "#discount")]
		(c2dom/val "#total" (-> (* quantity price)
			                (* (+ 1 (/ tax 100)))
							(- discount)
							(.toFixed 2)))))

	(defn reset-form []
		(let [fields ["#quantity" "#price" "#tax" "#discount" "#total"]
			  init ["1" "1.00" "0.0" "0.0" "0.00"]]
		  (dorun (map c2dom/val fields init))))

	(defn add-info [el name]
		(c2dom/append! el [:div {:id name} (str "Click to " name)]))

	(defn remove-info [el]
		(c2dom/remove! el))
```

which are slightly different since they use the *c2 library* functions
(actually those are not the only differences, we wanted to show other
possible implementations).

> As mentioned above, no ^:export tags must be provided.



[1]:  https://github.com/levand/domina
[2]:  https://github.com/teropa/hiccups
[3]:  https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-04.md
[4]:  http://www.larryullman.com/books/modern-javascript-develop-and-design/
[5]:  http://www.larryullman.com/
[6]:  https://raw.github.com/magomimmo/modern-cljs/tut-11/doc/images/form-idle.png
[7]:  https://raw.github.com/magomimmo/modern-cljs/tut-11/doc/images/form-events.png
[8]:  https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-06.md
[9]:  https://github.com/lynaghk/c2.git
[10]: https://github.com/lynaghk/c2/blob/master/README.markdown